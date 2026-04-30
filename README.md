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

### Go package with build

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: '1.23'

      - uses: aux4/test-package-action@v1
        with:
          build_command: aux4 builder linux
          system_packages: jq

  publish:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-go@v5
        with:
          go-version: '1.23'

      - name: Build all platforms
        run: |
          curl -fsSL https://aux4.sh | sh
          aux4 build

      - uses: aux4/publish-package-action@v1
        with:
          level: ${{ inputs.level || 'patch' }}
          aux4_token: ${{ secrets.AUX4_ACCESS_TOKEN }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Multi-platform testing

```yaml
jobs:
  test:
    runs-on: ${{ matrix.platform }}
    strategy:
      matrix:
        platform: [ubuntu-latest, macos-latest]
        include:
          - platform: ubuntu-latest
            build: aux4 builder linux
          - platform: macos-latest
            build: aux4 builder darwin
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: '1.23'

      - uses: aux4/test-package-action@v1
        with:
          build_command: ${{ matrix.build }}
          system_packages: jq
```

### With dependencies

```yaml
- uses: aux4/test-package-action@v1
  with:
    build_command: aux4 builder linux
    system_packages: jq curl
    npm_packages: typescript
    packages: aux4/config aux4/validator
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `working_directory` | Working directory containing the repository | No | `.` |
| `package_directory` | Directory containing the package `.aux4` file | No | `package` |
| `test_directory` | Directory containing the test files | No | `test` |
| `build_command` | Command to build the package before testing | No | - |
| `system_packages` | Space-separated system packages to install (apt/brew) | No | - |
| `npm_packages` | Space-separated global npm packages to install | No | - |
| `packages` | Space-separated aux4 packages to install | No | - |

## What it does

1. Installs `aux4` and `pkger`
2. Installs system packages via `apt` (Linux) or `brew` (macOS)
3. Installs global npm packages
4. Installs aux4 dependency packages
5. Installs `aux4/test` runner
6. Runs `build_command` (if provided)
7. Packages and installs the package under test
8. Runs `aux4 test run`

## Workflow

```
[test-package-action]          [publish-package-action]
        |                               |
  install aux4                     version bump
  install deps                     pkger build
  build (current platform)         pkger publish
  pkger build                      git tag
  pkger install                    gh release
  test run
```

## License

Apache-2.0
