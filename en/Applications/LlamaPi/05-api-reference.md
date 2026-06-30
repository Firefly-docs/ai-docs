# API Reference

This is an API reference paragraph.

## Core Classes

### `TestClass`

```python
class TestClass:
    def __init__(
        self,
        param_one: str,
        param_two: int = 123,
        param_three: float = 0.5,
        param_four: bool = False,
    ):
        """
        This is a docstring comment.

        Args:
            param_one: This is a parameter description
            param_two: This is a parameter description
            param_three: This is a parameter description
            param_four: This is a parameter description
        """
        ...
```

### `TestClass.run()`

```python
def run(
    self,
    input_text: str,
    max_length: int = 256,
    stream: bool = False,
) -> str | Generator[str, None, None]:
    """
    This is a docstring comment.

    Args:
        input_text: This is a parameter description
        max_length: This is a parameter description
        stream: This is a parameter description

    Returns:
        This is a return value description
    """
    ...
```

## Utility Functions

### `utility_func_a()`

```python
def utility_func_a(param_x: str, param_y: int) -> dict:
    """
    This is a docstring comment.
    """
    ...
```

### `utility_func_b()`

```python
def utility_func_b(param_x: list) -> str:
    """
    This is a docstring comment.
    """
    ...
```

## Exceptions

| Exception | Description |
|-----------|-------------|
| `TestErrorA` | This is an exception description |
| `TestErrorB` | This is an exception description |
| `TestErrorC` | This is an exception description |
