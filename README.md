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
docker-compose up -d

# Verify containers are running
docker ps

# Check container logs
docker-compose logs -f
```

### Step 3: Access Jenkins
1. Open your browser and navigate to `http://localhost:8080`
2. Get the initial admin password:
   ```bash
   docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Log in with:
   - Username: `admin`
   - Password: (from step 2)

### Step 4: Configure Jenkins

#### Initial Setup
1. Choose "Install suggested plugins"
2. Wait for installation to complete
3. Create admin user with secure password

#### Install Docker Plugins
1. Go to "Manage Jenkins" > "Manage Plugins"
2. Click on "Available" tab
3. Search for and install the following Docker plugins:
   - Docker Pipeline
   - Docker plugin
   - docker-build-step
4. Restart Jenkins after installation:
   ```bash
   # Restart Jenkins container
   docker-compose restart jenkins
   
   # Wait for Jenkins to start (about 30 seconds)
   # Then verify Jenkins is running
   docker-compose ps
   ```

#### Create Pipeline Project
1. On the Jenkins dashboard:
   - Click "New Item" (or "Create New Jobs")
   - Enter project name: `simple-python-pyinstaller-app`
   - Select "Pipeline"
   - Click "OK"

2. In the pipeline configuration:
   - Scroll down to "Pipeline" section
   - In "Definition" dropdown, select "Pipeline script from SCM"
   - In "SCM" dropdown, select "Git"
   - For "Repository URL", enter: `https://github.com/TarakKatoch/Jenkins-Orchestration.git`
   - For "Branch Specifier", enter: `*/master`
   - Click "Save"

3. After saving, Jenkins will:
   - Clone your repository
   - Look for the Jenkinsfile
   - Start the pipeline

### Step 5: Install Docker in Jenkins Container
```bash
# Install Docker in Jenkins container
docker-compose exec jenkins bash -c "apt-get update && apt-get install -y docker.io"

# Start Docker service
docker-compose exec jenkins bash -c "service docker start"

# Verify Docker installation
docker-compose exec jenkins docker --version
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

 1. Download the executable from Jenkins:
   - Go to Jenkins dashboard
   - Click on `simple-python-pyinstaller-app`
   - Click on the latest build number
   - Look for "Build Artifacts" section
   - Click on `add2vals` to download it to your local machine

2. Set up for local testing:
   - Create a `dist` folder in your local project directory (if it doesn't exist)
   - Place the downloaded `add2vals` executable in the `dist` folder

3. Test the executable:
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
