# Python Application with Jenkins CI/CD Pipeline

This project demonstrates a simple Python application with a complete CI/CD pipeline using Jenkins, Docker, and GitHub. The application provides a command-line tool that adds two numbers together, with automated testing and continuous integration/deployment.

## Project Overview

The project consists of a simple Python application that adds two numbers together, with automated testing and continuous integration/deployment using Jenkins. The application is packaged into a standalone executable using PyInstaller.

### Components

1. **Python Application**
   - `add2vals.py`: Main application file that handles command-line arguments and displays results
   - `calc.py`: Core calculation module containing the addition logic
   - `test_calc.py`: Test suite with unit tests for the calculation module
   - `requirements.txt`: Python dependencies with specific versions

2. **Jenkins Pipeline**
   - `Jenkinsfile`: Pipeline configuration defining the CI/CD workflow
   - `docker-compose.yml`: Jenkins and agent setup with volume mappings and network configuration

3. **Docker Configuration**
   - Jenkins container: Main Jenkins server with persistent storage
   - Jenkins agent container: For distributed builds
   - Python build container: For building and testing the application

## Prerequisites

### System Requirements
- Docker (version 20.10.0 or higher)
- Docker Compose (version 2.0.0 or higher)
- Git (version 2.39.0 or higher)
- At least 4GB RAM
- 20GB free disk space

### Software Dependencies
- Python 3.11 or higher
- pip (Python package installer)
- GitHub account with repository access
- Jenkins (will be installed via Docker)

## Required Jenkins Plugins

The following plugins are required for this project:

1. **Pipeline Plugins**
   - Pipeline
   - Pipeline: Stage Step
   - Pipeline: Basic Steps
   - Pipeline: SCM Step
   - Pipeline: Shared Groovy Libraries

2. **Git Integration**
   - Git
   - Git Client
   - GitHub Integration

3. **Docker Integration**
   - Docker Pipeline
   - Docker plugin
   - Docker API Plugin
   - docker-build-step

4. **Testing and Reporting**
   - JUnit Plugin
   - Test Results Analyzer

5. **UI and Usability**
   - Blue Ocean
   - Pipeline Stage View
   - Build Timeout

### Installing Plugins

1. Go to Jenkins Dashboard
2. Navigate to "Manage Jenkins" > "Manage Plugins"
3. Go to "Available" tab
4. Search for and install each plugin listed above
5. Restart Jenkins after installation

## Project Structure

```
.
├── sources/
│   ├── add2vals.py      # Main application entry point
│   ├── calc.py          # Core calculation module
│   └── test_calc.py     # Unit tests
├── test-reports/        # Generated test reports
├── dist/               # Compiled executables
├── requirements.txt    # Python dependencies
├── Jenkinsfile        # Pipeline configuration
├── docker-compose.yml # Docker setup
└── README.md         # This documentation
```

## Setup Instructions

### 1. Clone the Repository

```bash
# Clone the repository
git clone https://github.com/TarakKatoch/Jenkins-Orchestration.git

# Navigate to project directory
cd Jenkins-Orchestration

# Verify repository contents
ls -la
```

### 2. Start Jenkins

```bash
# Start Jenkins and required containers
docker compose up -d

# Verify containers are running
docker ps

# Check container logs
docker compose logs -f
```

### 3. Access Jenkins

1. Open your browser and navigate to `http://localhost:8080`
2. Get the initial admin password:
   ```bash
   docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Log in with:
   - Username: `admin`
   - Password: (from step 2)

### 4. Configure Jenkins

1. **Initial Setup**
   - Choose "Install suggested plugins"
   - Wait for installation to complete
   - Create admin user with secure password

2. **Configure GitHub Integration**
   - Go to "Manage Jenkins" > "Configure System"
   - Add GitHub server configuration
   - Test connection to GitHub

3. **Create Pipeline Project**
   - Click "New Item"
   - Enter project name: `simple-python-pyinstaller-app`
   - Select "Pipeline"
   - Configure Git repository:
     - Repository URL: `https://github.com/TarakKatoch/Jenkins-Orchestration.git`
     - Branch specifier: `*/master`
   - Save configuration

## Pipeline Stages

The Jenkins pipeline consists of four stages:

1. **Setup**
   ```groovy
   stage('Setup') {
       steps {
           sh '''
               # Update package list
               apt-get update
               
               # Install required system packages
               apt-get install -y binutils
               
               # Update pip and install Python dependencies
               python -m pip install --upgrade pip
               pip install -r requirements.txt
           '''
       }
   }
   ```

2. **Build**
   ```groovy
   stage('Build') {
       steps {
           // Compile Python files
           sh 'python -m py_compile sources/add2vals.py sources/calc.py'
           
           // Stash source files for later stages
           stash name: 'sources', includes: 'sources/**'
       }
   }
   ```

3. **Test**
   ```groovy
   stage('Test') {
       steps {
           // Run tests with JUnit reporting
           sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
           
           // Archive test results
           junit 'test-reports/results.xml'
       }
   }
   ```

4. **Deliver**
   ```groovy
   stage('Deliver') {
       steps {
           // Create standalone executable
           sh 'pyinstaller --onefile sources/add2vals.py'
       }
   }
   ```

## PyInstaller Configuration

PyInstaller is used to create a standalone executable from the Python application. This section explains how PyInstaller is configured and used in the project.

### Usage in Pipeline

The Jenkins pipeline uses PyInstaller in the Deliver stage:
```groovy
stage('Deliver') {
    steps {
        // Create standalone executable
        sh 'pyinstaller --onefile sources/add2vals.py'
    }
}
```

The `--onefile` flag creates a single executable that contains all dependencies.

### How It Works

1. **Analysis Phase**
   - PyInstaller analyzes `add2vals.py` and its dependencies
   - Identifies required Python files, libraries, and resources
   - Creates a dependency graph of the application

2. **Bundling Phase**
   - Packages all identified dependencies
   - Includes Python interpreter
   - Bundles required libraries and resources
   - Creates a single executable file

3. **Output**
   - Executable is created in the `dist` directory
   - Named `add2vals` (or `add2vals.exe` on Windows)
   - Contains everything needed to run the application

### Testing the Executable

After a successful pipeline run, the executable will be available in the `dist` directory:

1. **Access in Jenkins**
   - Go to the build page in Jenkins
   - Look for "Build Artifacts" section
   - Find `dist/add2vals.exe` (Windows) or `dist/add2vals` (Linux/macOS)
   - Click to download the executable

2. **Local Testing**
   ```bash
   # Navigate to dist directory
   cd dist

   # Test with integer inputs
   ./add2vals 5 3  # Should output: 8.0

   # Test with decimal inputs
   ./add2vals 10.5 7.3  # Should output: 17.8

   # Test with negative numbers
   ./add2vals -5 3  # Should output: -2.0
   ```

Note: The executable name and extension will depend on your operating system:
- Windows: `add2vals.exe`
- Linux/macOS: `add2vals`

### Benefits

1. **Portability**
   - Single executable file
   - No Python installation required
   - Easy distribution

2. **Dependency Management**
   - All dependencies included
   - No external package installation needed
   - Consistent environment

3. **Cross-Platform Support**
   - Creates platform-specific executables
   - Works on Windows, Linux, and macOS
   - Native performance

### Troubleshooting PyInstaller

If you encounter issues with PyInstaller:

1. **Missing Dependencies**
   ```bash
   # Check if all dependencies are in requirements.txt
   pip freeze > requirements.txt
   ```

2. **Import Errors**
   - Verify all imports are explicit
   - Check for dynamic imports
   - Ensure all modules are included

3. **File Not Found**
   - Check file paths are relative
   - Verify resource files are included
   - Use `--add-data` flag if needed

4. **Performance Issues**
   - Consider using `--onedir` instead of `--onefile`
   - Profile the application
   - Check for unnecessary imports

### Customizing PyInstaller

You can customize PyInstaller behavior using spec files or command-line options:

```bash
# Create spec file
pyi-makespec --onefile sources/add2vals.py

# Build using spec file
pyinstaller add2vals.spec

# Additional options
pyinstaller --onefile --windowed --icon=app.ico sources/add2vals.py
```

Common options:
- `--onefile`: Create single executable
- `--windowed`: Hide console window (Windows)
- `--icon`: Set application icon
- `--name`: Specify output name
- `--add-data`: Include additional files

## Testing the Application

### Running Tests Locally

```bash
# Install dependencies
pip install -r requirements.txt

# Run tests
py.test sources/test_calc.py -v

# Run tests with coverage report
py.test --cov=sources sources/test_calc.py
```

### Testing the Executable

After a successful pipeline run, the executable will be available in the `dist` directory:

```bash
# Navigate to dist directory
cd dist

# Test with integer inputs
./add2vals 5 3  # Should output: 8.0

# Test with decimal inputs
./add2vals 10.5 7.3  # Should output: 17.8

# Test with negative numbers
./add2vals -5 3  # Should output: -2.0
```

## Troubleshooting

### Common Issues

1. **Jenkins Container Issues**
   ```bash
   # Stop all containers
   docker compose down
   
   # Remove volumes (optional, will delete Jenkins data)
   docker compose down -v
   
   # Start containers again
   docker compose up -d
   
   # Check container status
   docker ps
   
   # View container logs
   docker compose logs -f jenkins
   ```

2. **Docker Permission Issues**
   ```bash
   # Install Docker in Jenkins container
   docker compose exec jenkins bash -c "apt-get update && apt-get install -y docker.io"
   
   # Start Docker service
   docker compose exec jenkins bash -c "service docker start"
   
   # Verify Docker installation
   docker compose exec jenkins docker --version
   ```

3. **Build Failures**
   - Check Jenkins logs for specific error messages
   - Verify all dependencies are installed
   - Ensure GitHub repository is accessible
   - Check Docker container permissions
   - Verify network connectivity

### Log Files

Important log files and their locations:
- Jenkins logs: `/var/jenkins_home/logs/`
- Docker logs: `docker compose logs`
- Pipeline logs: Available in Jenkins UI under each build

## Maintenance

### Updating Dependencies

1. Modify `requirements.txt` with new versions
2. Commit changes:
   ```bash
   git add requirements.txt
   git commit -m "Update dependencies"
   git push
   ```
3. Jenkins will automatically rebuild with new dependencies

### Adding New Features

1. Create new feature branch:
   ```bash
   git checkout -b feature/new-feature
   ```

2. Make changes and add tests
3. Run local tests:
   ```bash
   py.test sources/test_calc.py
   ```

4. Commit and push changes:
   ```bash
   git add .
   git commit -m "Add new feature"
   git push origin feature/new-feature
   ```

5. Create pull request on GitHub
6. Jenkins will run pipeline on pull request

## Security Considerations

1. **Jenkins Security**
   - Credentials are stored in Docker volumes
   - Regular security updates
   - Access control and user management

2. **GitHub Integration**
   - Secure webhooks
   - Token-based authentication
   - Repository access control

3. **Docker Security**
   - Containers run with limited permissions
   - Regular image updates
   - Network isolation

4. **Application Security**
   - Input validation
   - Error handling
   - Secure file operations

## Future Improvements

1. **Pipeline Enhancements**
   - Add deployment stages
   - Implement versioning
   - Add code quality checks
   - Set up email notifications

2. **Environment Configuration**
   - Configure different environments (dev/staging/prod)
   - Environment-specific settings
   - Deployment strategies

3. **Monitoring and Logging**
   - Add performance monitoring
   - Enhanced logging
   - Alert system

4. **Testing Improvements**
   - Add integration tests
   - Performance testing
   - Security testing

## Contributing

1. Fork the repository
2. Create feature branch:
   ```bash
   git checkout -b feature/your-feature
   ```
3. Make changes and add tests
4. Run tests locally
5. Commit changes:
   ```bash
   git commit -m "Add your feature"
   ```
6. Push to your fork:
   ```