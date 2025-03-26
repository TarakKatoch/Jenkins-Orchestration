# Simple Python PyInstaller App

This is a simple Python application that demonstrates CI/CD with Jenkins. The application provides a command-line tool `add2vals` that adds two values or concatenates them if they are strings.

## Features

- Command-line interface for adding two values
- Supports both numeric addition and string concatenation
- Unit tests with pytest
- Automated build and test pipeline with Jenkins
- PyInstaller packaging for standalone executable

## Prerequisites

- Python 3.x
- pip (Python package installer)
- Jenkins (for CI/CD pipeline)

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd simple-python-pyinstaller-app
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

## Usage

The application can be used in two ways:

1. As a Python script:
```bash
python sources/add2vals.py <value1> <value2>
```

2. As a standalone executable (after building):
```bash
./dist/add2vals <value1> <value2>
```

### Examples

```bash
# Numeric addition
python sources/add2vals.py 5 3
# Output: 8

# String concatenation
python sources/add2vals.py "Hello" "World"
# Output: HelloWorld

# Mixed types
python sources/add2vals.py "Number" 42
# Output: Number42
```

## Development

### Running Tests

```bash
pytest sources/test_calc.py
```

### Building with PyInstaller

```bash
pyinstaller --onefile sources/add2vals.py
```

## Jenkins Pipeline

The project includes a Jenkinsfile that defines a CI/CD pipeline with three stages:

1. Build: Compiles the Python source files
2. Test: Runs the unit tests and generates JUnit reports
3. Deliver: Creates a standalone executable using PyInstaller

The pipeline will automatically run when changes are pushed to the repository. 