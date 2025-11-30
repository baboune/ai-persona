## Helm Chart Testing Best Practices

### ConfigMap Testing Patterns

When testing Helm chart ConfigMaps that contain YAML configuration
files (using the `config.yaml: |` multi-line string format), use
these testing patterns:

#### Pattern 1: Full Config Validation (Recommended for comprehensive tests)

Use the `equal` assertion with a multi-line string to validate the
entire config.yaml content. This ensures the complete configuration
matches expectations.

**Example:**

```yaml
- it: renders a default config
  asserts:
    - equal:
        path: data["config.yaml"]
        value: |
          kafka:
            bootStrapServer: kafka.example.com:9092
            useIam: false
          logging:
            level: "INFO"
```

**When to use:**

- Testing default configuration values
- Validating complete configuration structure
- Ensuring no unexpected fields are added
- Comprehensive regression testing

**Reference examples in codebase:**

- `substate/hussar/chart/tests/configmap_test.yaml`
- `substate/thoth/server/chart/tests/configmap_test.yaml`

#### Pattern 2: Specific Field Validation (For targeted tests)

Use `matchRegex` to validate specific fields within the config.yaml
string without testing the entire content.

**Example:**

```yaml
- it: check fanOut option
  asserts:
    - matchRegex:
        path: data["config.yaml"]
        pattern: 'sdrFanOutOnTypeAndTenant:\s+false'
```

**When to use:**

- Testing specific configuration overrides
- Validating dynamic values (cluster size, custom settings)
- Testing conditional configuration sections
- Quick validation of critical fields

#### Why Not Use Nested Paths?

ConfigMaps with multi-line string content (using `|`) store the
configuration as a **string**, not as parsed YAML. Therefore:

❌ **This will NOT work:**

```yaml
- equal:
    path: data["config.yaml"].sink.sdrFanOutOnTypeAndTenant
    value: false
```

Error: `unknown path data["config.yaml"].sink.sdrFanOutOnTypeAndTenant`

✅ **Use one of the patterns above instead**

#### Test Structure Recommendation

For each ConfigMap, include:

1. Basic validation tests (isKind, namespace)
2. One comprehensive "renders a default config" test (Pattern 1)
3. Specific tests for configurable values (Pattern 2)

**Example test file structure:**

```yaml
templates:
  - configmap.yaml
tests:
  - it: is configmap
    asserts:
      - isKind:
          of: ConfigMap
  - it: sets namespace
    asserts:
      - equal:
          path: metadata.namespace
          value: NAMESPACE
  - it: renders a default config
    asserts:
      - equal:
          path: data["config.yaml"]
          value: |
            # Full expected config here
  - it: validates custom setting
    set:
      someValue: "custom"
    asserts:
      - matchRegex:
          path: data["config.yaml"]
          pattern: 'someField:\s+"custom"'
```
