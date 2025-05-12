# Using REST APIs

> "The design of the Web is a network of networks... Web technologies are thus built
> on the principle of working together with other technologies via universal protocols
> and formats."
>
> —— W3C TAG, 2004

In today's interconnected digital world, applications rarely exist in isolation. They need to communicate with other services, fetch data from remote sources, and integrate with other systems. This is where APIs (Application Programming Interfaces) come in, and specifically REST (Representational State Transfer) APIs, which have become the dominant paradigm for web services communication.

In this chapter, we'll explore how to interact with REST APIs using Python's `requests` library. We'll start with the fundamentals and then move on to a practical example of copying Jira tickets between cloud instances—a common task in enterprise environments when migrating projects or creating test environments with real data.

## Understanding REST APIs

REST is an architectural style for designing networked applications. RESTful services use HTTP requests to perform CRUD (Create, Read, Update, Delete) operations. Here are the key elements of REST APIs:

- **Resources**: Everything is a resource, identified by a unique URI
- **HTTP Methods**: Standard HTTP methods like GET, POST, PUT, DELETE are used to operate on resources
- **Representations**: Resources can have multiple representations, usually JSON or XML
- **Stateless**: Each request from client to server must contain all information needed to understand and complete the request

### Common HTTP Methods

REST APIs typically use these HTTP methods:

| Method | Purpose                                    |
|--------|------------------------------------------- |
| GET    | Retrieve a resource                        |
| POST   | Create a new resource                      |
| PUT    | Update a resource (or create if it doesn't exist) |
| PATCH  | Partially update a resource                |
| DELETE | Remove a resource                          |

### Understanding API Documentation

Before using any API, you need to understand how it works. Good API documentation typically includes:

- Authentication methods
- Available endpoints
- Required and optional parameters
- Expected response formats and status codes
- Rate limits

The Jira REST API documentation, for example, is extensive and well-organized, making it a good case study for understanding how to work with a production-grade API.

## Getting Started with the requests Library

The `requests` library is the de facto standard for making HTTP requests in Python. If you haven't installed it yet, you can do so with pip:

```bash
pip install requests
```

Let's start with some basic examples:

```python
import requests

# Simple GET request
response = requests.get('https://api.github.com')

# Check status code
print(f"Status code: {response.status_code}")

# Print response content
print(response.json())
```

The `requests` library makes it easy to work with different HTTP methods:

```python
# GET request with parameters
params = {'key1': 'value1', 'key2': 'value2'}
response = requests.get('https://httpbin.org/get', params=params)

# POST request with JSON data
data = {'name': 'John', 'age': 30}
response = requests.post('https://httpbin.org/post', json=data)

# PUT request
response = requests.put('https://httpbin.org/put', json=data)

# DELETE request
response = requests.delete('https://httpbin.org/delete')
```

### Handling Headers and Authentication

Many APIs require authentication. The `requests` library makes it easy to include headers and authentication credentials:

```python
# Including custom headers
headers = {
    'User-Agent': 'Python Requests App',
    'Accept': 'application/json',
}
response = requests.get('https://api.github.com/user', headers=headers)

# Basic authentication
response = requests.get(
    'https://api.example.com/protected',
    auth=('username', 'password')
)

# OAuth token authentication
headers = {'Authorization': 'Bearer YOUR_ACCESS_TOKEN'}
response = requests.get('https://api.example.com/me', headers=headers)
```

### Handling Responses

The `requests` library provides convenient methods for working with responses:

```python
response = requests.get('https://api.github.com')

# Check if request was successful
if response.status_code == 200:
    print("Success!")
elif response.status_code == 404:
    print("Not found!")
else:
    print(f"Error: {response.status_code}")

# Convert response to JSON
data = response.json()
print(data)

# Access response headers
content_type = response.headers['Content-Type']
print(f"Content type: {content_type}")

# Access raw content
raw = response.content
print(raw)
```

## Working with the Jira REST API

Now let's dive into working with the Jira REST API, which is a good example of a real-world, production-grade API. We'll build functionality to copy tickets from one Jira instance to another—a common task when migrating between projects or creating test instances with real data.

### Authentication

Jira Cloud offers several authentication options. For our example, we'll use Basic Authentication with an API token:

```python
import requests
from requests.auth import HTTPBasicAuth
import json

# Initialize credentials for source and target instances
source_email = "your-email@example.com"
source_api_token = "your-source-api-token"
source_base_url = "https://your-source-instance.atlassian.net"

target_email = "your-email@example.com"
target_api_token = "your-target-api-token"
target_base_url = "https://your-target-instance.atlassian.net"

# Create auth objects
source_auth = HTTPBasicAuth(source_email, source_api_token)
target_auth = HTTPBasicAuth(target_email, target_api_token)

# Set headers
headers = {
    "Accept": "application/json",
    "Content-Type": "application/json"
}
```

### Getting an Issue from Source Jira

Let's fetch an issue from the source Jira instance:

```python
def get_issue(issue_key):
    """Fetch an issue from the source Jira instance."""
    api_endpoint = f"{source_base_url}/rest/api/3/issue/{issue_key}"
    
    response = requests.get(
        api_endpoint,
        auth=source_auth,
        headers=headers
    )
    
    if response.status_code != 200:
        print(f"Error getting issue {issue_key}: {response.status_code}")
        print(response.text)
        return None
    
    return response.json()
```

### Creating an Issue in Target Jira

Now let's create a new issue in the target Jira instance:

```python
def create_issue(project_key, summary, description, issue_type="Task"):
    """Create a new issue in the target Jira instance."""
    api_endpoint = f"{target_base_url}/rest/api/3/issue"
    
    payload = {
        "fields": {
            "project": {
                "key": project_key
            },
            "summary": summary,
            "description": {
                "type": "doc",
                "version": 1,
                "content": [
                    {
                        "type": "paragraph",
                        "content": [
                            {
                                "type": "text",
                                "text": description
                            }
                        ]
                    }
                ]
            },
            "issuetype": {
                "name": issue_type
            }
        }
    }
    
    response = requests.post(
        api_endpoint,
        auth=target_auth,
        headers=headers,
        data=json.dumps(payload)
    )
    
    if response.status_code not in [200, 201]:
        print(f"Error creating issue: {response.status_code}")
        print(response.text)
        return None
    
    return response.json()
```

### Copying an Issue Between Instances

Now let's combine these functions to copy an issue from one Jira instance to another:

```python
def copy_issue(source_issue_key, target_project_key):
    """Copy an issue from source to target Jira instance."""
    # Get the source issue
    source_issue = get_issue(source_issue_key)
    if not source_issue:
        return None
    
    # Extract relevant fields
    summary = source_issue["fields"]["summary"]
    
    # Handle potential differences in description format
    description = ""
    if "description" in source_issue["fields"] and source_issue["fields"]["description"]:
        # For Jira Cloud, description might be in Atlassian Document Format
        if isinstance(source_issue["fields"]["description"], dict) and "content" in source_issue["fields"]["description"]:
            # This is a simplification - a real implementation would need to parse ADF properly
            description = "Original description in ADF format - see source issue"
        else:
            description = source_issue["fields"]["description"]
    
    issue_type = source_issue["fields"]["issuetype"]["name"]
    
    # Create the issue in the target instance
    new_issue = create_issue(target_project_key, summary, description, issue_type)
    
    if new_issue:
        print(f"Successfully copied {source_issue_key} to {new_issue['key']}")
        return new_issue["key"]
    else:
        print(f"Failed to copy {source_issue_key}")
        return None
```

### Handling Attachments

Jira issues often have attachments that we might want to copy as well:

```python
def get_attachments(issue_key):
    """Get all attachments for an issue."""
    issue_data = get_issue(issue_key)
    if not issue_data or "fields" not in issue_data or "attachment" not in issue_data["fields"]:
        return []
    
    return issue_data["fields"]["attachment"]

def download_attachment(attachment):
    """Download an attachment from source Jira."""
    response = requests.get(
        attachment["content"],
        auth=source_auth,
        headers={"Accept": "application/octet-stream"}
    )
    
    if response.status_code != 200:
        print(f"Error downloading attachment: {response.status_code}")
        return None
    
    return response.content

def upload_attachment(issue_key, filename, content):
    """Upload an attachment to target Jira."""
    api_endpoint = f"{target_base_url}/rest/api/3/issue/{issue_key}/attachments"
    
    headers_with_attachment = headers.copy()
    headers_with_attachment["X-Atlassian-Token"] = "no-check"
    del headers_with_attachment["Content-Type"]  # Let requests set the correct multipart content type
    
    files = {
        "file": (filename, content)
    }
    
    response = requests.post(
        api_endpoint,
        auth=target_auth,
        headers=headers_with_attachment,
        files=files
    )
    
    if response.status_code not in [200, 201]:
        print(f"Error uploading attachment: {response.status_code}")
        print(response.text)
        return False
    
    return True

def copy_attachments(source_issue_key, target_issue_key):
    """Copy all attachments from source to target issue."""
    attachments = get_attachments(source_issue_key)
    if not attachments:
        print(f"No attachments found for {source_issue_key}")
        return
    
    for attachment in attachments:
        print(f"Copying attachment: {attachment['filename']}")
        content = download_attachment(attachment)
        if content:
            success = upload_attachment(target_issue_key, attachment["filename"], content)
            if success:
                print(f"Successfully copied attachment {attachment['filename']}")
            else:
                print(f"Failed to copy attachment {attachment['filename']}")
```

### Copying Comments

Comments are another important part of Jira issues:

```python
def get_comments(issue_key):
    """Get all comments for an issue."""
    api_endpoint = f"{source_base_url}/rest/api/3/issue/{issue_key}/comment"
    
    response = requests.get(
        api_endpoint,
        auth=source_auth,
        headers=headers
    )
    
    if response.status_code != 200:
        print(f"Error getting comments for {issue_key}: {response.status_code}")
        return []
    
    data = response.json()
    return data.get("comments", [])

def add_comment(issue_key, comment_body):
    """Add a comment to an issue in the target Jira."""
    api_endpoint = f"{target_base_url}/rest/api/3/issue/{issue_key}/comment"
    
    # For Jira Cloud, comments need to be in Atlassian Document Format
    payload = {
        "body": {
            "type": "doc",
            "version": 1,
            "content": [
                {
                    "type": "paragraph",
                    "content": [
                        {
                            "type": "text",
                            "text": comment_body
                        }
                    ]
                }
            ]
        }
    }
    
    response = requests.post(
        api_endpoint,
        auth=target_auth,
        headers=headers,
        data=json.dumps(payload)
    )
    
    if response.status_code not in [200, 201]:
        print(f"Error adding comment: {response.status_code}")
        print(response.text)
        return False
    
    return True

def copy_comments(source_issue_key, target_issue_key):
    """Copy all comments from source to target issue."""
    comments = get_comments(source_issue_key)
    if not comments:
        print(f"No comments found for {source_issue_key}")
        return
    
    for comment in comments:
        # Extract plain text from Atlassian Document Format
        comment_text = "Original comment by: " + comment.get("author", {}).get("displayName", "Unknown")
        
        if isinstance(comment.get("body"), dict) and "content" in comment["body"]:
            # This is a simplified extraction - a real implementation would parse ADF properly
            comment_text += "\n\nOriginal comment in ADF format - see source issue"
        else:
            comment_text += "\n\n" + str(comment.get("body", ""))
        
        success = add_comment(target_issue_key, comment_text)
        if success:
            print(f"Successfully copied comment")
        else:
            print(f"Failed to copy comment")
```

### Putting It All Together

Now let's combine these functions to create a complete script for copying a Jira issue with its attachments and comments:

```python
def copy_complete_issue(source_issue_key, target_project_key):
    """Copy a complete issue with attachments and comments."""
    # Copy the basic issue
    target_issue_key = copy_issue(source_issue_key, target_project_key)
    if not target_issue_key:
        return None
    
    # Copy attachments
    copy_attachments(source_issue_key, target_issue_key)
    
    # Copy comments
    copy_comments(source_issue_key, target_issue_key)
    
    print(f"Issue successfully copied from {source_issue_key} to {target_issue_key}")
    return target_issue_key

# Example usage
if __name__ == "__main__":
    import sys
    
    if len(sys.argv) != 3:
        print("Usage: python jira_copy.py SOURCE_ISSUE_KEY TARGET_PROJECT_KEY")
        sys.exit(1)
    
    source_issue_key = sys.argv[1]
    target_project_key = sys.argv[2]
    
    copy_complete_issue(source_issue_key, target_project_key)
```

## Best Practices for Working with REST APIs

Working with REST APIs requires careful attention to several aspects. Here are some best practices to follow:

### Rate Limiting and Pagination

Many APIs impose rate limits to prevent abuse. Be mindful of these limits and implement appropriate handling:

```python
def get_all_issues_in_project(project_key):
    """Get all issues in a project, handling pagination."""
    start_at = 0
    max_results = 50
    total = None
    all_issues = []
    
    while total is None or start_at < total:
        api_endpoint = f"{source_base_url}/rest/api/3/search"
        params = {
            "jql": f"project={project_key}",
            "startAt": start_at,
            "maxResults": max_results
        }
        
        response = requests.get(
            api_endpoint,
            auth=source_auth,
            headers=headers,
            params=params
        )
        
        if response.status_code != 200:
            print(f"Error getting issues: {response.status_code}")
            break
        
        data = response.json()
        total = data["total"]
        issues = data["issues"]
        all_issues.extend(issues)
        start_at += max_results
        
        # Implement a delay to respect rate limits
        import time
        time.sleep(1)
    
    return all_issues
```

### Error Handling

Robust error handling is essential when working with external APIs:

```python
def make_api_request(method, url, **kwargs):
    """Make an API request with error handling."""
    try:
        response = requests.request(method, url, **kwargs)
        response.raise_for_status()  # Raises an exception for 4XX/5XX responses
        return response
    except requests.exceptions.HTTPError as err:
        print(f"HTTP Error: {err}")
        if response.text:
            print(f"Response: {response.text}")
    except requests.exceptions.ConnectionError as err:
        print(f"Connection Error: {err}")
    except requests.exceptions.Timeout as err:
        print(f"Timeout Error: {err}")
    except requests.exceptions.RequestException as err:
        print(f"Request Error: {err}")
    
    return None
```

### Authentication and Security

Always handle authentication credentials securely:

1. Never hardcode credentials in your script
2. Use environment variables or secure credential stores
3. Implement proper token refresh mechanisms for OAuth

```python
import os

# Get credentials from environment variables
source_email = os.environ.get("JIRA_SOURCE_EMAIL")
source_api_token = os.environ.get("JIRA_SOURCE_TOKEN")
source_base_url = os.environ.get("JIRA_SOURCE_URL")

if not all([source_email, source_api_token, source_base_url]):
    print("Error: Source Jira credentials not set in environment variables")
    sys.exit(1)
```

### Caching

For frequently accessed data that doesn't change often, consider implementing caching:

```python
import functools
from datetime import datetime, timedelta

# Simple time-based cache
cache = {}

def cached_request(expiry_seconds=300):
    """Decorator for caching API requests."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(url, *args, **kwargs):
            if url in cache:
                timestamp, data = cache[url]
                if datetime.now() - timestamp < timedelta(seconds=expiry_seconds):
                    return data
            
            result = func(url, *args, **kwargs)
            cache[url] = (datetime.now(), result)
            return result
        return wrapper
    return decorator

@cached_request(expiry_seconds=60)
def get_project_info(project_key):
    """Get project info with caching."""
    api_endpoint = f"{source_base_url}/rest/api/3/project/{project_key}"
    
    response = requests.get(
        api_endpoint,
        auth=source_auth,
        headers=headers
    )
    
    if response.status_code != 200:
        print(f"Error getting project info: {response.status_code}")
        return None
    
    return response.json()
```

### Retries

Network requests can fail for various reasons. Implementing retries can improve resilience:

```python
import time
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_session_with_retries(retries=3, backoff_factor=0.3):
    """Create a requests session with retry capability."""
    session = requests.Session()
    retry = Retry(
        total=retries,
        backoff_factor=backoff_factor,
        status_forcelist=[429, 500, 502, 503, 504],
        allowed_methods=["GET", "POST", "PUT", "DELETE", "PATCH"]
    )
    adapter = HTTPAdapter(max_retries=retry)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    return session

# Use the session for requests
session = create_session_with_retries()
response = session.get(
    f"{source_base_url}/rest/api/3/issue/{issue_key}",
    auth=source_auth,
    headers=headers
)
```

## Advanced Techniques

### Asynchronous Requests

For performance-critical applications or when making many API calls, consider using asynchronous requests with libraries like `aiohttp` or `httpx`:

```python
import asyncio
import aiohttp

async def fetch_issue(session, issue_key):
    """Fetch a single issue asynchronously."""
    api_endpoint = f"{source_base_url}/rest/api/3/issue/{issue_key}"
    auth = aiohttp.BasicAuth(source_email, source_api_token)
    
    async with session.get(api_endpoint, auth=auth, headers=headers) as response:
        if response.status != 200:
            print(f"Error getting issue {issue_key}: {response.status}")
            return None
        
        return await response.json()

async def fetch_multiple_issues(issue_keys):
    """Fetch multiple issues concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_issue(session, key) for key in issue_keys]
        return await asyncio.gather(*tasks)

# Run the async function
issue_data = asyncio.run(fetch_multiple_issues(["PROJ-1", "PROJ-2", "PROJ-3"]))
```

### Using Webhooks

For real-time updates, consider using webhooks if the API supports them:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/webhook/jira', methods=['POST'])
def jira_webhook():
    """Handle Jira webhook events."""
    event_data = request.json
    
    # Check the event type
    event_type = event_data.get('webhookEvent')
    
    if event_type == 'jira:issue_created':
        # Handle issue creation
        issue_key = event_data.get('issue', {}).get('key')
        print(f"Issue created: {issue_key}")
        # Process the new issue...
    
    elif event_type == 'jira:issue_updated':
        # Handle issue update
        issue_key = event_data.get('issue', {}).get('key')
        print(f"Issue updated: {issue_key}")
        # Process the updated issue...
    
    return jsonify({'status': 'success'})

if __name__ == '__main__':
    app.run(port=5000)
```

### API Clients and Libraries

For popular APIs, consider using dedicated client libraries:

```python
from jira import JIRA

# Initialize JIRA client
source_jira = JIRA(
    server=source_base_url,
    basic_auth=(source_email, source_api_token)
)

target_jira = JIRA(
    server=target_base_url,
    basic_auth=(target_email, target_api_token)
)

# Get an issue
issue = source_jira.issue('PROJ-123')

# Create an issue
new_issue = target_jira.create_issue(
    project='TARG',
    summary=issue.fields.summary,
    description=issue.fields.description,
    issuetype={'name': issue.fields.issuetype.name}
)
```

## Summary

In this chapter, we've explored how to work with REST APIs using Python's `requests` library, with a focus on the Jira REST API as a practical example. We've learned how to:

- Make HTTP requests using different methods (GET, POST, PUT, DELETE)
- Handle authentication, headers, and JSON data
- Copy Jira issues between instances, including attachments and comments
- Implement best practices like rate limiting, error handling, and caching
- Use advanced techniques like asynchronous requests and webhooks

REST APIs are a fundamental building block of modern web applications, allowing different systems to communicate and integrate with each other. The patterns and techniques discussed in this chapter apply not just to Jira but to any REST API you might work with, from social media platforms to cloud services, financial systems, and more.

## Exercises

1. Extend the Jira issue copying script to handle custom fields specific to your organization.

2. Modify the script to copy an entire project from one Jira instance to another, preserving issue relationships.

3. Create a simple REST API client for another service you use regularly (e.g., GitHub, Trello, or a weather service).

4. Implement a rate limiting decorator that ensures your application respects API rate limits.

5. Build a simple webhook server that listens for Jira events and triggers appropriate actions. 