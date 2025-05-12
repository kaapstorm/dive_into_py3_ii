# Testing with pytest

> "The act of writing tests is not testing; it is expressing a
requirement of the code in an executable form."
>
> —— Kevlin Henney

If you've ever worked on a Python project of any significant size, you know that testing is essential. Automated tests help ensure that your code behaves as expected, catch regressions before they reach production, and serve as living documentation of how your code is meant to be used. While Python's standard library includes the `unittest` module, many Python developers prefer [pytest](https://docs.pytest.org/en/latest/) for its simplicity, powerful features, and extensibility.

In this chapter, we'll explore some advanced pytest features that can help you write more maintainable, comprehensive tests with less code. Specifically, we'll look at how to use pytest's parametrization capability to run the same test with multiple sets of inputs, and how to use the [pytest-unmagic](https://github.com/dimagi/pytest-unmagic) plugin to make fixture dependencies more explicit.

## Testing with Parametrized Tests

One of the most powerful features of pytest is the ability to run the same test function with different sets of inputs. This is called *parametrization*, and it allows you to cover many test cases without duplicating test code. Let's see how this works with a simple example:

```python
# test_math.py
import pytest

def add(a, b):
    return a + b

@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (0, 0, 0),
    (-1, 1, 0),
    (100, -50, 50)
])
def test_add(a, b, expected):
    result = add(a, b)
    assert result == expected
```

When you run this test file with pytest, it will execute the `test_add` function four times, once for each tuple of parameters. The first parameter to `@pytest.mark.parametrize` is a string containing the names of the parameters, separated by commas. The second parameter is a list of tuples, each containing the values to pass to the test function.

This is much more concise than writing four separate test functions, and it makes it easy to add new test cases. If you find a bug with a specific input, you can add it to the list of parameter tuples without writing a new test function.

### Parametrizing with Complex Data Types

You can use any Python data types as parameters, including lists, dictionaries, and objects:

```python
# test_data_processing.py
import pytest

def process_data(data_dict):
    result = {}
    for key, value in data_dict.items():
        result[key] = value * 2
    return result

@pytest.mark.parametrize("input_dict, expected", [
    ({"a": 1, "b": 2}, {"a": 2, "b": 4}),
    ({}, {}),
    ({"x": 0}, {"x": 0}),
    ({"a": -1}, {"a": -2})
])
def test_process_data(input_dict, expected):
    result = process_data(input_dict)
    assert result == expected
```

### Adding IDs to Parametrized Tests

When you have many test cases or complex inputs, it can be hard to identify which one failed from the test output. You can add custom IDs to your parametrized tests to make this easier:

```python
# test_with_ids.py
import pytest

def is_palindrome(text):
    text = text.lower().replace(" ", "")
    return text == text[::-1]

@pytest.mark.parametrize("text, expected", [
    ("racecar", True),
    ("hello", False),
    ("A man a plan a canal Panama", True),
    ("", True)
], ids=["simple_palindrome", "non_palindrome", "complex_palindrome", "empty_string"])
def test_is_palindrome(text, expected):
    assert is_palindrome(text) == expected
```

When you run this test, the output will include the custom IDs instead of the parameter values, making it easier to identify which test case failed.

### Parametrizing Multiple Arguments Independently

Sometimes you want to test all combinations of multiple parameters. You can do this by stacking multiple `@pytest.mark.parametrize` decorators:

```python
# test_combinations.py
import pytest

def multiply_then_add(a, b, c):
    return a * b + c

@pytest.mark.parametrize("a", [1, 2, 3])
@pytest.mark.parametrize("b", [10, 20])
@pytest.mark.parametrize("c", [0, 1, -1])
def test_multiply_then_add(a, b, c):
    result = multiply_then_add(a, b, c)
    expected = a * b + c
    assert result == expected
```

This will run the test function for all combinations of `a`, `b`, and `c`, resulting in 18 (3×2×3) test executions.

### Parametrizing Fixtures

In addition to parametrizing test functions directly, you can also parametrize fixtures. This is useful when you need to set up a complex environment for your tests and want to run the same tests with different environments.

```python
# test_database.py
import pytest

class Database:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connected = False
        
    def connect(self):
        # In a real implementation, this would connect to the database
        self.connected = True
        return self.connected
        
    def disconnect(self):
        self.connected = False

@pytest.fixture(params=["postgresql://localhost:5432/test", "mysql://localhost:3306/test"])
def db(request):
    connection_string = request.param
    database = Database(connection_string)
    database.connect()
    yield database
    database.disconnect()

def test_database_connection(db):
    assert db.connected == True
```

In this example, the `db` fixture is parametrized with two different connection strings. The test function `test_database_connection` will be run twice, once with each database connection.

## Using pytest-unmagic for Explicit Fixtures

One aspect of pytest that some developers find challenging is the way that fixtures are magically injected into tests based on parameter names. While this can make tests concise, it can also make it harder to understand where fixtures are coming from, especially in large codebases with many fixtures. The pytest-unmagic plugin provides a way to make fixture dependencies more explicit, while still leveraging the power of pytest fixtures.

Let's look at how pytest-unmagic works compared to standard pytest fixtures.

### Using unmagic.fixture

With pytest-unmagic, you define fixtures using the `unmagic.fixture` decorator and apply them to tests using the `unmagic.use` decorator:

```python
# test_unmagic.py
from unmagic import fixture, use

traces = []

@fixture
def tracer():
    assert not traces, f"unexpected traces before setup: {traces}"
    yield
    traces.clear()

@use(tracer)
def test_append():
    traces.append("hello")
    assert traces, "expected at least one trace"
```

The key difference here is that the fixture is explicitly applied to the test function with the `@use` decorator, rather than being implicitly injected based on parameter names. This makes it immediately clear which fixtures a test depends on, even without looking at the parameter list.

### Fetching Fixture Values

If you need to access the value of a fixture, you can call the fixture within the test function:

```python
# test_unmagic_values.py
from unmagic import fixture, use

@fixture
def config():
    yield {"api_url": "https://api.example.com", "timeout": 30}

@use(config)
def test_config():
    cfg = config()  # Get the fixture value by calling it
    assert "api_url" in cfg
    assert "timeout" in cfg
```

### Applying Fixtures to Classes

You can apply fixtures to entire test classes, which will apply them to all test methods in the class:

```python
# test_unmagic_class.py
from unmagic import fixture, use

@fixture
def database():
    db = {"users": [], "products": []}
    yield db
    db.clear()

@use(database)
class TestDatabase:
    def test_add_user(self):
        db = database()
        db["users"].append({"id": 1, "name": "Alice"})
        assert len(db["users"]) == 1
        
    def test_add_product(self):
        db = database()
        db["products"].append({"id": 1, "name": "Widget"})
        assert len(db["products"]) == 1
```

### Combining with Standard pytest Fixtures

One of the strengths of pytest-unmagic is that it works alongside standard pytest fixtures. You can use both in the same test:

```python
# test_mixed.py
import pytest
from unmagic import fixture, use, get_request

@fixture
def app_config():
    yield {"debug": True}

@use(app_config)
def test_with_capsys():
    capsys = get_request().getfixturevalue("capsys")
    cfg = app_config()
    
    print(f"Debug mode: {cfg['debug']}")
    
    captured = capsys.readouterr()
    assert captured.out == "Debug mode: True\n"
```

In this example, we're using our unmagic fixture `app_config` alongside the standard pytest fixture `capsys`, which we retrieve using `get_request().getfixturevalue()`.

### Benefits of Explicit Fixtures

Using explicit fixtures with pytest-unmagic has several benefits:

1. **Clarity**: It's immediately clear which fixtures a test depends on.
2. **Refactoring safety**: If you rename a fixture, you'll get a Python error rather than a silent test failure.
3. **Linting compatibility**: You avoid linter warnings about redefining outer scope names.
4. **Import tracking**: IDEs and linters can properly track fixture imports and warn about unused imports.

## Best Practices for pytest Tests

Whether you're using parametrized tests, the pytest-unmagic plugin, or just standard pytest features, here are some best practices for writing good pytest tests:

### Keep Tests Simple

Each test should test one thing and have a clear purpose. If you find yourself writing a test that verifies multiple behaviors, split it into multiple tests.

```python
# Better: separate tests for each behavior
def test_user_creation():
    user = create_user("alice", "password123")
    assert user.username == "alice"
    
def test_user_authentication():
    user = create_user("alice", "password123")
    assert authenticate("alice", "password123") == True
    assert authenticate("alice", "wrong") == False
```

### Use Descriptive Test Names

Your test names should describe what they're testing, not just reference the function they're testing. This makes it easier to understand test failures.

```python
# Less clear
def test_process_data():
    # ...

# More clear
def test_process_data_doubles_all_values():
    # ...
```

### Organize Tests with Classes

For related tests, consider grouping them in a class. This can make your test files more organized and allow you to share fixtures among related tests.

```python
class TestUserAuthentication:
    def test_successful_login(self):
        # ...
        
    def test_failed_login_wrong_password(self):
        # ...
        
    def test_failed_login_user_not_found(self):
        # ...
```

### Use Fixtures for Setup and Teardown

Fixtures are a powerful way to handle test setup and teardown. They can make your tests more concise and avoid duplication.

```python
@pytest.fixture
def authenticated_user():
    user = create_user("alice", "password123")
    login_user(user)
    yield user
    logout_user(user)
    
def test_access_protected_resource(authenticated_user):
    result = access_resource("/protected")
    assert result.status_code == 200
```

### Test Edge Cases

Make sure to test edge cases and error conditions, not just the happy path.

```python
@pytest.mark.parametrize("username, password, expected_error", [
    ("", "password123", "Username cannot be empty"),
    ("alice", "", "Password cannot be empty"),
    ("a" * 101, "password123", "Username cannot exceed 100 characters")
])
def test_user_creation_validation_errors(username, password, expected_error):
    with pytest.raises(ValidationError) as excinfo:
        create_user(username, password)
    assert str(excinfo.value) == expected_error
```

## Summary

In this chapter, we've explored two powerful testing techniques with pytest: parametrized tests and explicit fixtures with the pytest-unmagic plugin. Parametrized tests allow you to run the same test with multiple sets of inputs, which can greatly reduce test code duplication and make it easier to add new test cases. The pytest-unmagic plugin provides a way to make fixture dependencies more explicit, which can improve test clarity and avoid some common pitfalls of pytest's standard fixture mechanism.

By combining these techniques with good testing practices, you can create a test suite that is comprehensive, maintainable, and provides confidence in your code's correctness.

## Exercises

1. Write a parametrized test for a function that validates email addresses. Include test cases for valid emails, emails with invalid formats, and edge cases.

2. Create a fixture that sets up a temporary database with some test data, and write a test that uses this fixture.

3. Refactor an existing test to use pytest-unmagic for its fixtures. Compare the two versions and reflect on the differences.

4. Write a test that uses multiple parametrized fixtures to test a function with multiple inputs.

5. Create a test class with multiple test methods that share a fixture. Implement this using both the standard pytest approach and the pytest-unmagic approach. 