# Python UV Tutorial: Modern Dependency Management

## Table of Contents
1. [Introduction to UV](#introduction-to-uv)
2. [Installation](#installation)
3. [Managing Environments](#managing-environments)
4. [Installing Packages](#installing-packages)
5. [Managing Dependencies](#managing-dependencies)
6. [Working with requirements.txt](#working-with-requirementstxt)
7. [UV vs Other Tools](#uv-vs-other-tools)
8. [Performance Benchmarks](#performance-benchmarks)
9. [Best Practices](#best-practices)
10. [Common Issues & Solutions](#common-issues--solutions)

## Introduction to UV

UV (pronounced "you-vee") is a modern Python packaging tool designed to be significantly faster than traditional tools like pip. It's written in Rust and aims to solve common pain points in Python dependency management:

- **Speed**: Much faster than pip, particularly for large dependency trees
- **Reliability**: Better dependency resolution with fewer conflicts
- **Compatibility**: Works with existing Python package formats and standards
- **Simplicity**: Streamlined command-line interface

UV combines the functionality of several tools:
- Package installation (like pip)
- Virtual environment management (like venv/virtualenv)
- Dependency resolution (like pip-tools)

## Installation

### System Requirements
- Python 3.7+
- Compatible with Windows, macOS, and Linux

### Installing UV

#### Using pip
```bash
pip install uv
```

#### Using pipx (recommended)
```bash
pipx install uv
```

#### Using Homebrew (macOS)
```bash
brew install uv
```

#### Verify Installation
```bash
uv --version
```

## Managing Environments

UV includes built-in virtual environment management, simplifying your workflow by combining package and environment management.

### Creating a Virtual Environment

```bash
# Create a virtual environment in the .venv directory
uv venv

# Create a virtual environment with a specific name
uv venv /path/to/custom/env

# Create with a specific Python version
uv venv --python=python3.10
```

### Activating a Virtual Environment

```bash
# Windows
.venv\Scripts\activate

# Unix/macOS
source .venv/bin/activate
```

### Deactivating a Virtual Environment

```bash
deactivate
```

## Installing Packages

UV offers a faster alternative to pip for package installation.

### Basic Package Installation

```bash
# Install a package
uv pip install requests

# Install multiple packages
uv pip install pandas matplotlib scikit-learn

# Install a specific version
uv pip install flask==2.0.1

# Install with version constraints
uv pip install "django>=4.0,<5.0"
```

### Installing in Development Mode

```bash
# Install the current directory in development mode
uv pip install -e .
```

### Installing from Git

```bash
uv pip install git+https://github.com/username/repo.git
```

### Installing Extras

```bash
uv pip install "package[extra1,extra2]"
```

## Managing Dependencies

UV shines in dependency management with clearer resolution and fewer conflicts.

### Project Dependencies with pyproject.toml

```bash
# Create a virtual environment and install dependencies from pyproject.toml
uv venv
uv pip install -e .
```

### Syncing Dependencies

```bash
# Install or update all dependencies to match requirements
uv pip sync requirements.txt
```

### Compiling Requirements

UV can compile a requirements.txt with locked versions from higher-level dependencies:

```bash
# Generate a requirements.txt from pyproject.toml
uv pip compile pyproject.toml -o requirements.txt

# Include development dependencies
uv pip compile pyproject.toml --extra=dev -o dev-requirements.txt
```

## Working with requirements.txt

UV works well with traditional requirements.txt files but processes them much faster.

### Installing from requirements.txt

```bash
# Install packages from requirements.txt
uv pip install -r requirements.txt

# Update all packages to the latest versions
uv pip install --upgrade -r requirements.txt
```

### Creating/Updating requirements.txt

```bash
# Freeze current environment to requirements.txt
uv pip freeze > requirements.txt

# Compile a requirements file from high-level dependencies
uv pip compile requirements.in -o requirements.txt
```

## UV vs Other Tools

### UV vs pip

| Feature | UV | pip |
|---------|----|----|
| Speed | 5-100x faster | Baseline |
| Dependency Resolution | Advanced | Basic |
| Virtual Environments | Integrated | Requires virtualenv |
| Lockfiles | Native support | Requires pip-tools |
| Package Format Support | Wheels, sdists | Wheels, sdists |
| Maturity | Newer | Well-established |

### UV vs Poetry/PDM

| Feature | UV | Poetry/PDM |
|---------|----|----|
| Speed | Very fast | Moderate |
| Project Configuration | Works with any | Specific format |
| Learning Curve | Gentle | Steeper |
| Lockfile Format | Standard | Custom |
| Publishing Packages | Basic | Advanced |

## Performance Benchmarks

UV is consistently faster than traditional tools, especially for large dependency trees:

| Operation | UV | pip |
|-----------|----|----|
| Install Django | ~1 second | ~5 seconds |
| Install 100 packages | ~5 seconds | ~45 seconds |
| Create virtual environment | ~0.5 seconds | ~2 seconds |
| Resolve dependencies | ~2 seconds | ~20 seconds |

*Note: Performance varies by system and package size*

## Best Practices

### Project Structure

```
my_project/
├── .venv/                 # UV-created virtual environment
├── pyproject.toml         # Project metadata and dependencies
├── requirements.txt       # Locked dependencies (compiled)
├── requirements-dev.txt   # Development dependencies
├── src/                   # Source code
└── tests/                 # Tests
```

### Recommended Workflow

1. **Project Setup**
   ```bash
   # Initialize project with virtual environment
   mkdir my_project && cd my_project
   uv venv
   source .venv/bin/activate  # Or .venv\Scripts\activate on Windows
   ```

2. **Dependency Management**
   ```bash
   # Create a pyproject.toml file (can be done manually or with a tool)
   # Then install your project in development mode
   uv pip install -e .
   
   # Or work with requirements.txt
   uv pip install -r requirements.txt
   ```

3. **Locking Dependencies**
   ```bash
   # Generate lockfile from pyproject.toml
   uv pip compile pyproject.toml -o requirements.txt
   ```

4. **Updating Dependencies**
   ```bash
   # Update all dependencies
   uv pip compile --upgrade pyproject.toml -o requirements.txt
   
   # Apply the updates
   uv pip sync requirements.txt
   ```

### CI/CD Integration

```yaml
# Example GitHub Actions workflow
name: Python Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install UV
        run: pip install uv
      - name: Install dependencies
        run: |
          uv venv
          source .venv/bin/activate
          uv pip install -r requirements.txt
          uv pip install -r requirements-dev.txt
      - name: Run tests
        run: |
          source .venv/bin/activate
          pytest
```

## Common Issues & Solutions

### Common Error: Package Not Found

**Problem:** UV can't find a package.

**Solution:** Check package name and PyPI availability. Try with `--index-url` for custom package repositories:

```bash
uv pip install package-name --index-url https://custom-pypi.org/simple
```

### Common Error: Dependency Conflicts

**Problem:** Conflicting dependencies between packages.

**Solution:** UV usually resolves these better than pip, but you can:

```bash
# Get more detailed information
uv pip install package-name --verbose

# Try looser version constraints
uv pip install "package-name>=1.0" instead of "package-name==1.0.2"
```

### Common Error: SSL Certificate Verification

**Problem:** SSL certificate verification fails.

**Solution:**
```bash
# Temporarily bypass (not recommended for production)
uv pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org package-name

# Better: Fix your certificates
```

### Speed Optimization

To get the maximum speed benefit from UV:

1. Use wheels whenever possible
2. Use UV's sync command for installing known dependencies
3. Consider setting up a local PyPI cache for frequently used packages

## Conclusion

UV represents a significant advancement in Python package management, offering speed improvements that can dramatically improve development workflows. As a drop-in replacement for many pip commands with backward compatibility, it's easy to adopt incrementally.

While still evolving, UV has already proven valuable for projects dealing with complex dependency trees or requiring faster builds. Its simplicity and focus on speed make it a compelling alternative to traditional Python packaging tools.

For the latest updates and features, check the official documentation and GitHub repository.