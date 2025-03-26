# Python Application with Jenkins CI/CD Pipeline

## 1. Project Overview

This project demonstrates a simple Python application with a complete CI/CD pipeline using Jenkins, Docker, and GitHub. The application provides a command-line tool that adds two numbers together, with automated testing and continuous integration/deployment.

## 2. System Requirements

- Docker (version 20.10.0 or higher)
- Docker Compose (version 2.0.0 or higher)
- Git (version 2.39.0 or higher)
- At least 4GB RAM
- 20GB free disk space
- Python 3.11 or higher
- pip (Python package installer)
- GitHub account with repository access

## 3. Components

### Project Structure
```
.
├── sources/
│   ├── add2vals.py      # Main application entry point
│   ├── calc.py          # Core calculation module
│   └── test_calc.py     # Unit tests
├── dist/               # Compiled executables
├── requirements.txt    # Python dependencies
├── Jenkinsfile        # Pipeline configuration
├── docker-compose.yml # Docker setup
└── README.md         # This documentation
```

### Python Application
- `add2vals.py`: Main application file that handles command-line arguments and displays results
- `calc.py`: Core calculation module containing the addition logic
- `test_calc.py`: Test suite with unit tests for the calculation module
- `requirements.txt`: Python dependencies with specific versions

### Jenkins Pipeline
- `Jenkinsfile`: Pipeline configuration defining the CI/CD workflow
- `docker-compose.yml`: Jenkins and agent setup with volume mappings and network configuration

### Docker Configuration
- Jenkins container: Main Jenkins server with persistent storage
- Jenkins agent container: For distributed builds
- Python build container: For building and testing the application

## 4. Steps to Perform This Project

### Step 1: Clone the Repository
```bash
# Clone the repository
git clone https://github.com/TarakKatoch/Jenkins-Orchestration.git

# Navigate to project directory
cd Jenkins-Orchestration

# Verify repository contents
ls -la
```

### Step 2: Start Jenkins
```bash
# Start Jenkins and required containers
docker compose up -d

# Verify containers are running
docker ps

# Check container logs
docker compose logs -f
```

### Step 3: Access Jenkins
1. Open your browser and navigate to `http://localhost:8080`
2. Get the initial admin password:
   ```bash
   docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Log in with:
   - Username: `admin`
   - Password: (from step 2)

### Step 4: Configure Jenkins

#### Initial Setup
1. Choose "Install suggested plugins"
2. Wait for installation to complete
3. Create admin user with secure password

#### Configure GitHub Integration
1. Go to "Manage Jenkins" > "Configure System"
2. Add GitHub server configuration
3. Test connection to GitHub

#### Create Pipeline Project
1. Click "New Item"
2. Enter project name: `simple-python-pyinstaller-app`
3. Select "Pipeline"
4. Configure Git repository:
   - Repository URL: `https://github.com/TarakKatoch/Jenkins-Orchestration.git`
   - Branch specifier: `*/master`
5. Save configuration

### Step 5: Install Docker in Jenkins Container
```bash
# Install Docker in Jenkins container
docker compose exec jenkins bash -c "apt-get update && apt-get install -y docker.io"

# Start Docker service
docker compose exec jenkins bash -c "service docker start"

# Verify Docker installation
docker compose exec jenkins docker --version
```

## 5. Pipeline Stages

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

## 6. Build Pipeline

The build pipeline is triggered automatically when changes are pushed to the GitHub repository. Here's how it works:

### Pipeline Trigger
1. **GitHub Webhook**
   - When code is pushed to GitHub
   - Webhook notifies Jenkins
   - Pipeline starts automatically

2. **Manual Trigger**
   - Go to Jenkins dashboard
   - Select your pipeline project
   - Click "Build Now"

### Pipeline Execution Flow

1. **Checkout Stage**
   ```groovy
   checkout scm
   ```
   - Clones the repository
   - Checks out the specified branch
   - Prepares workspace

2. **Setup Stage**
   - Updates package list
   - Installs system dependencies
   - Sets up Python environment
   - Installs project dependencies

3. **Build Stage**
   - Compiles Python files
   - Creates build artifacts
   - Stashes files for later stages

4. **Test Stage**
   - Runs unit tests
   - Generates test reports
   - Archives test results

5. **Deliver Stage**
   - Creates executable
   - Archives build artifacts
   - Makes files available for download

### Pipeline Status

1. **Success Indicators**
   - All stages complete successfully
   - Tests pass
   - Executable is generated
   - Artifacts are archived

2. **Failure Indicators**
   - Stage fails
   - Tests fail
   - Build errors
   - Missing dependencies

### Pipeline Logs

1. **Accessing Logs**
   - Click on build number
   - Select "Console Output"
   - View detailed execution logs

2. **Troubleshooting Logs**
   - Check for error messages
   - Verify command execution
   - Review test results
   - Check dependency installation

### Pipeline Artifacts

1. **Generated Files**
   - Executable in `dist/` directory
   - Test reports in `test-reports/`
   - Build logs
   - Test results

2. **Accessing Artifacts**
   - Go to build page
   - Click "Artifacts"
   - Download required files

## 7. How to Test After Build

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

## 8. How It Works

1. **Source Code Management**
   - Code is stored in GitHub repository
   - Jenkins pulls code on each build
   - Changes trigger automatic builds

2. **Build Process**
   - Jenkins creates isolated build environment
   - Installs required dependencies
   - Compiles Python code
   - Runs automated tests

3. **Testing**
   - Unit tests verify functionality
   - Test results are archived
   - Build fails if tests fail

4. **Delivery**
   - Creates standalone executable
   - Archives build artifacts
   - Makes executable available for download

## 9. What Does PyInstaller Do?

PyInstaller is used to create a standalone executable from the Python application. Here's how it works:

1. **Analysis Phase**
   - Analyzes `add2vals.py` and its dependencies
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

### Benefits of PyInstaller
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