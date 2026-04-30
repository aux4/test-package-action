# Test aux4 Package Action

A GitHub Action to run tests for aux4 packages before publishing. Supports both Linux and macOS runners.

## Usage

### Basic example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aux4/test-package-action@v1
        with:
          package_directory: package
```

### With dependencies

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aux4/test-package-action@v1
        with:
          system_packages: jq
          npm_packages: typescript
          packages: aux4/config aux4/validator
```

### Multi-platform testing

```yaml
jobs:
  test:
    runs-on: ${{ matrix.platform }}
    strategy:
      matrix:
        platform: [ubuntu-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: '1.23'

      - name: Build
        run: |
          go build -o package/dist/$(go env GOOS)/$(go env GOARCH)/my-binary .

      - uses: aux4/test-package-action@v1
        with:
          packages: aux4/config

  publish:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Build all platforms
        run: ...

      - uses: aux4/publish-package-action@v1
        with:
          level: ${{ inputs.level || 'patch' }}
          aux4_token: ${{ secrets.AUX4_ACCESS_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `working_directory` | Working directory containing the repository | No | `.` |
| `package_directory` | Directory containing the package `.aux4` file | No | `package` |
| `test_directory` | Directory containing the test files | No | `test` |
| `system_packages` | Space-separated system packages to install (apt/brew) | No | - |
| `npm_packages` | Space-separated global npm packages to install | No | - |
| `packages` | Space-separated aux4 packages to install | No | - |

## What it does

1. Installs `aux4` and `pkger`
2. Installs system packages via `apt` (Linux) or `brew` (macOS)
3. Installs global npm packages
4. Installs aux4 dependency packages
5. Installs `aux4/test` runner
6. Builds and installs the package under test
7. Runs `aux4 test run`

## Workflow

```
[Your Build Step] -> [test-package-action] -> [publish-package-action]
     |                      |                         |
  go build              install deps              version bump
  npm build             pkger build               pkger publish
  cargo build           pkger install             git tag
  etc.                  test run                  gh release
```

## License

Apache-2.0
