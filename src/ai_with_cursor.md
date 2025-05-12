# AI-Assisted Programming with Cursor

> The best way to predict the future is to invent it.
>
> —Alan Kay

## Diving In

```python
def calculate_fibonacci(n: int) -> list[int]:
    """Calculate Fibonacci sequence up to n terms."""
    if n <= 0:
        return []
    elif n == 1:
        return [0]
    
    sequence = [0, 1]
    while len(sequence) < n:
        sequence.append(sequence[-1] + sequence[-2])
    return sequence

# Let's test it
print(calculate_fibonacci(10))
```

Save this as `fibonacci.py` and run it from the command line:

```
$ python fibonacci.py
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

This code calculates the Fibonacci sequence, but what if you're not sure how to implement it? Or what if you want to add error handling, documentation, or type hints? This is where AI-assisted programming with Cursor comes in.

## What is Cursor?

Cursor is a modern code editor that integrates AI capabilities directly into your development workflow. It's built on top of VS Code and combines the best features of traditional IDEs with powerful AI assistance. The key features that make Cursor stand out are:

1. **AI Chat**: An interactive chat interface that understands your codebase
2. **AI Code Generation**: Ability to generate code based on natural language descriptions
3. **AI Code Editing**: Smart suggestions for improving and refactoring code
4. **AI Rules**: Customizable rules for maintaining code quality and consistency

## Getting Started with Cursor

To begin using Cursor:

1. Download and install Cursor from [cursor.com](https://www.cursor.com)
2. Open your Python project in Cursor
3. Use `Cmd/Ctrl + K` to open the AI chat interface
4. Start asking questions about your code or request assistance

Let's see how Cursor can help us improve our Fibonacci example:

```python
def calculate_fibonacci(n: int) -> list[int]:
    """
    Calculate the Fibonacci sequence up to n terms.
    
    Args:
        n (int): The number of terms to calculate
        
    Returns:
        list[int]: A list containing the Fibonacci sequence
        
    Raises:
        ValueError: If n is negative
    """
    if n < 0:
        raise ValueError("n must be non-negative")
    if n == 0:
        return []
    if n == 1:
        return [0]
    
    sequence = [0, 1]
    while len(sequence) < n:
        sequence.append(sequence[-1] + sequence[-2])
    return sequence
```

Cursor's AI can help you:
- Add comprehensive documentation
- Implement error handling
- Add type hints
- Suggest optimizations
- Write tests

## Using AI Rules in Cursor

One of Cursor's most powerful features is AI Rules. These are customizable guidelines that help maintain code quality and consistency across your project. Here's how to use them:

1. Open the Command Palette (`Cmd/Ctrl + Shift + P`)
2. Type "AI Rules" and select "Create New AI Rule"
3. Define your rule using natural language

Example rules you might create:

- "All functions must have type hints and docstrings"
- "Use f-strings for string formatting"
- "Follow PEP 8 style guidelines"
- "Include error handling for all file operations"

The AI will then help enforce these rules as you write code.

## Best Practices for AI-Assisted Programming

When working with Cursor's AI features, keep these best practices in mind:

1. **Be Specific**: The more specific your questions or requests, the better the AI can help
2. **Review Suggestions**: Always review AI-generated code before accepting it
3. **Iterate**: Use the AI to iteratively improve your code
4. **Learn**: Use the AI's explanations to understand why certain approaches are recommended

## Common Use Cases

### Code Generation

```python
# Ask Cursor: "Generate a function to validate email addresses"
def is_valid_email(email: str) -> bool:
    """
    Validate an email address format.
    
    Args:
        email (str): The email address to validate
        
    Returns:
        bool: True if the email is valid, False otherwise
    """
    import re
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))
```

### Code Refactoring

```python
# Ask Cursor: "Refactor this code to use list comprehension"
# Before:
squares = []
for i in range(10):
    squares.append(i ** 2)

# After:
squares = [i ** 2 for i in range(10)]
```

### Documentation

```python
# Ask Cursor: "Add comprehensive documentation to this function"
def process_data(data: list[dict], key: str) -> list:
    """
    Process a list of dictionaries by extracting values for a specific key.
    
    Args:
        data (list[dict]): List of dictionaries to process
        key (str): The key to extract from each dictionary
        
    Returns:
        list: List of values for the specified key
        
    Raises:
        KeyError: If the specified key is not found in any dictionary
        TypeError: If data is not a list or contains non-dictionary items
    """
    return [item[key] for item in data]
```

## Debugging with AI

Cursor's AI can help you debug code by:

1. Analyzing error messages
2. Suggesting fixes
3. Explaining why errors occur
4. Providing alternative approaches

```python
# Ask Cursor: "Why is this code raising a TypeError?"
def process_numbers(numbers):
    return sum(numbers) / len(numbers)

# The AI might suggest:
def process_numbers(numbers: list[float]) -> float:
    """
    Calculate the average of a list of numbers.
    
    Args:
        numbers (list[float]): List of numbers to process
        
    Returns:
        float: The average of the numbers
        
    Raises:
        ValueError: If the list is empty
        TypeError: If the list contains non-numeric values
    """
    if not numbers:
        raise ValueError("Cannot process empty list")
    return sum(numbers) / len(numbers)
```

## Testing and Self-Correction

Cursor's AI capabilities extend to testing and self-correction, making it easier to write and maintain reliable code. Let's explore how to use Cursor for test-driven development and automated code improvement.

### Writing Tests with AI

```python
# Ask Cursor: "Write tests for the calculate_fibonacci function"
import pytest

def test_fibonacci_sequence():
    """Test basic Fibonacci sequence generation."""
    assert calculate_fibonacci(0) == []
    assert calculate_fibonacci(1) == [0]
    assert calculate_fibonacci(5) == [0, 1, 1, 2, 3]
    assert calculate_fibonacci(10) == [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

def test_fibonacci_negative_input():
    """Test handling of negative input."""
    with pytest.raises(ValueError):
        calculate_fibonacci(-1)

def test_fibonacci_large_input():
    """Test handling of large input."""
    result = calculate_fibonacci(100)
    assert len(result) == 100
    assert result[0] == 0
    assert result[1] == 1
    # Verify Fibonacci property
    for i in range(2, len(result)):
        assert result[i] == result[i-1] + result[i-2]
```

### Running Tests in Cursor

Cursor makes it easy to run tests and see results:

1. Open the test file in Cursor
2. Use `Cmd/Ctrl + Shift + P` to open the Command Palette
3. Type "Run Tests" and select the appropriate test runner (pytest, unittest, etc.)
4. View test results in the integrated terminal

You can also use keyboard shortcuts:
- `Cmd/Ctrl + Shift + T` to run all tests
- `Cmd/Ctrl + Shift + R` to run the current test file
- `Cmd/Ctrl + Shift + D` to run the test at the current cursor position

### Self-Correction Based on Test Results

When tests fail, Cursor's AI can help analyze the failure and suggest fixes:

```python
# Original code with a bug
def calculate_fibonacci(n: int) -> list[int]:
    """Calculate Fibonacci sequence up to n terms."""
    if n <= 0:
        return []
    if n == 1:
        return [0]

    sequence = [0, 1]
    while len(sequence) < n:
        # Bug: Using wrong indices
        sequence.append(sequence[-2] + sequence[-3])
    return sequence

# After running tests, Cursor's AI might suggest:
def calculate_fibonacci(n: int) -> list[int]:
    """Calculate Fibonacci sequence up to n terms."""
    if n <= 0:
        return []
    if n == 1:
        return [0]

    sequence = [0, 1]
    while len(sequence) < n:
        # Fixed: Using correct indices
        sequence.append(sequence[-1] + sequence[-2])
    return sequence
```

### Continuous Testing and Improvement

Cursor can help maintain code quality through continuous testing:

1. **Test Coverage Analysis**: Ask Cursor to analyze test coverage and suggest additional test cases
2. **Edge Case Detection**: The AI can identify potential edge cases that need testing
3. **Performance Testing**: Get suggestions for performance improvements based on test results
4. **Regression Testing**: Automatically detect and fix regressions when code changes

Example of using Cursor for test coverage analysis:

```python
# Ask Cursor: "What test cases are missing for this function?"
def process_user_data(user_data: dict) -> dict:
    """Process user data and return sanitized version."""
    return {
        'name': user_data.get('name', '').strip(),
        'email': user_data.get('email', '').lower(),
        'age': int(user_data.get('age', 0))
    }

# Cursor might suggest these additional test cases:
def test_process_user_data():
    """Test user data processing."""
    # Test empty input
    assert process_user_data({}) == {'name': '', 'email': '', 'age': 0}

    # Test with whitespace
    assert process_user_data({'name': '  John  '}) == {'name': 'John', 'email': '', 'age': 0}

    # Test email case normalization
    assert process_user_data({'email': 'User@Example.com'}) == {'name': '', 'email': 'user@example.com', 'age': 0}

    # Test age conversion
    assert process_user_data({'age': '25'}) == {'name': '', 'email': '', 'age': 25}

    # Test invalid age
    with pytest.raises(ValueError):
        process_user_data({'age': 'invalid'})
```

## Conclusion

AI-assisted programming with Cursor represents a significant shift in how we write and maintain code. By combining the power of AI with a modern code editor, Cursor helps developers:

- Write better code faster
- Maintain consistent coding standards
- Learn best practices
- Debug more effectively
- Document code more thoroughly

Remember that AI is a tool to enhance your programming capabilities, not replace them. The best results come from using AI as a collaborative partner in your development process.

## Further Reading

- [Cursor Documentation](https://cursor.com/docs)
- [Python Type Hints](https://docs.python.org/3/library/typing.html)
- [PEP 8 Style Guide](https://peps.python.org/pep-0008/)
- [Python Testing with pytest](https://docs.pytest.org/) 