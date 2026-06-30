# Usage Guide

This is a usage guide paragraph.

## Basic Usage

```python
from test_module import TestClass

# This is a comment
instance = TestClass(param_one="test value")
result = instance.run("This is an input text")
print(result)
```

## Advanced Usage

```python
from test_module import TestClass, AdvancedFeature

# This is a comment
instance = TestClass(
    param_one="test value",
    param_two=42,
    param_three=True,
)

# This is a comment
for output in instance.stream("This is an input text"):
    print(output, end="")
```

## CLI Usage

```bash
test-command --flag-a value-a --flag-b value-b --input "This is a test input"
```

## Input Formats

| Format | Description | Example |
|--------|-------------|---------|
| Format A | This is a format description | `This is an example` |
| Format B | This is a format description | `This is an example` |

## Output Format

```json
{
  "field_a": "This is value A",
  "field_b": 123,
  "field_c": true
}
```
