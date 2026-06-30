# Configuration

This is a configuration guide paragraph.

## Config File

Config file path: `/path/to/config.yaml`

```yaml
# This is a comment
config_key_a: "This is config value A"
config_key_b: 123
config_key_c: true
config_key_d:
  - list item one
  - list item two
```

## Config Reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `config_key_a` | `str` | `"default_a"` | This is a config description |
| `config_key_b` | `int` | `123` | This is a config description |
| `config_key_c` | `bool` | `false` | This is a config description |
| `config_key_d` | `list` | `[]` | This is a config description |
| `config_key_e` | `float` | `0.5` | This is a config description |

## Environment Variables

```bash
export TEST_VAR_A="This is value A"
export TEST_VAR_B="This is value B"
```

## Examples

### Basic Config

```yaml
config_key_a: "basic value"
config_key_b: 456
```

### Advanced Config

```yaml
config_key_a: "advanced value"
config_key_b: 789
config_key_c: true
config_key_d:
  - advanced item one
  - advanced item two
```
