# Jenkins xUML Compilation Pipeline

## Overview
This project demonstrates how to set up a Jenkins pipeline for automated xUML model compilation and code generation, following the [official Scheer PAS documentation](https://doc.scheer-pas.com/bridge/24.1/continuous-delivery-with-the-bridge).

## Project Structure
```
otsjenkins/
├── Jenkinsfile              # Jenkins pipeline definition
├── tools/
│   └── xumlc-7.20.0.jar    # xUML compiler tool
├── URL/
│   ├── uml/
│   │   └── urlUrl.xml      # xUML model file
│   ├── testcase/           # Test cases for different adapters
│   │   ├── UrlAdapter/     # URL adapter test cases
│   │   ├── FTPExample/     # FTP adapter test cases
│   │   └── LDAPExampl/     # LDAP adapter test cases
│   └── regressiontest/     # Regression test suite
│       └── testsuite/
│           ├── testsuite.xml
│           └── testsuite.errors
└── build/                  # Generated output (created during build)
```

## Setup Instructions

### 1. Prerequisites
- Jenkins server with Git plugin installed
- Java runtime environment
- Git repository access

### 2. Jenkins Pipeline Configuration
The `Jenkinsfile` contains:
```groovy
pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '1'))
        disableConcurrentBuilds()
    }
    
    parameters {
        choice(name: 'XUMLC', choices: 'tools/xumlc-7.20.0.jar', description: 'Location of the xUML Compiler')
        choice(name: 'TEST_MODE', choices: ['basic', 'regression', 'full'], description: 'Testing mode to run')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jprocero-otse2esoftengr/otsjenkins.git'
            }
        }

        stage('Build') {
            steps {
                bat '''
                    echo Compiling xUML model...
                    mkdir build
                    java -jar tools/xumlc-7.20.0.jar -uml URL/uml/urlUrl.xml
                '''
                archiveArtifacts artifacts: 'build/**/*', fingerprint: true
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') { /* Basic validation tests */ }
                stage('Integration Tests') { /* Adapter integration tests */ }
                stage('Regression Tests') { /* Full regression suite */ }
            }
        }
        
        stage('Test Results Analysis') {
            steps {
                // Analyze and summarize test results
            }
        }
    }
}
```

### 3. Key Components
- **xUML Compiler**: `tools/xumlc-7.20.0.jar`
- **Model File**: `URL/uml/urlUrl.xml`
- **Test Cases**: Various adapter test scenarios
- **Output Directory**: `build/` (created during compilation)

### 4. Build Process
1. **Checkout**: Clones the repository
2. **Compile**: Runs xUML compiler on the model
3. **Generate**: Creates Java code and artifacts in `build/` directory
4. **Archive**: Automatically saves generated files as build artifacts
5. **Test**: Runs automated tests based on selected mode
6. **Analyze**: Generates test results summary

## Testing Modes (Level 3 CI/CD)

### Test Mode Selection
The pipeline supports three testing modes controlled by the `TEST_MODE` parameter:

#### 1. **Basic Mode** (`basic`)
- **Unit Tests**: Validates xUML compilation and basic functionality
- **Scope**: Minimal testing for quick feedback
- **Duration**: Fastest execution time

#### 2. **Regression Mode** (`regression`)
- **Unit Tests**: Basic validation tests
- **Integration Tests**: Tests adapter functionality (URL, FTP, LDAP)
- **Scope**: Comprehensive adapter testing
- **Duration**: Medium execution time

#### 3. **Full Mode** (`full`)
- **Unit Tests**: Basic validation tests
- **Integration Tests**: All adapter integration tests
- **Regression Tests**: Complete regression test suite validation
- **Scope**: Complete testing coverage
- **Duration**: Full execution time

### Test Types

#### Unit Tests
- **Purpose**: Validate basic compilation and generation
- **Checks**: 
  - Java files generated successfully
  - Compilation artifacts exist
  - Basic model validation

#### Integration Tests
- **Purpose**: Test adapter functionality and integration
- **Checks**:
  - URL Adapter test cases exist and are valid
  - FTP Example test cases are present
  - LDAP Example test cases are available
  - Test case structure validation

#### Regression Tests
- **Purpose**: Validate regression test suite configuration
- **Checks**:
  - Regression test suite exists (`testsuite.xml`)
  - Test suite configuration validation
  - Error file analysis (`testsuite.errors`)

## Artifact Archiving

### What is Artifact Archiving?
Artifact archiving automatically saves and preserves important files generated during the build process for later access and download.

### What Gets Archived
- **Generated Java files** from your UML model
- **Configuration files** (.rep files)
- **Test Results**: All test execution results and summaries
- **Documentation** (if generated)
- **Log files** from compilation and testing

### Benefits
- **Easy Access**: Team members can download artifacts directly from Jenkins
- **Deployment Ready**: Archived artifacts can be deployed to test/production environments
- **Historical Tracking**: Keep artifacts from previous builds for comparison
- **Debugging**: Access generated files and test results even after workspace cleanup

### How to Access Archived Artifacts
1. **In Jenkins UI**:
   - Go to your build
   - Click "Build Artifacts" in the left sidebar
   - Download any files you need

2. **Programmatically**:
   - Other Jenkins jobs can use archived artifacts
   - Deployment scripts can reference them

## Pipeline Configuration Options

### Build Discarder
```groovy
buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '1'))
```
- Keeps last 10 build logs
- Keeps artifacts from last 1 build
- Saves disk space while preserving recent history

### Concurrent Build Prevention
```groovy
disableConcurrentBuilds()
```
- Prevents multiple builds from running simultaneously
- Ensures build consistency and resource management

### Parameters
```groovy
parameters {
    choice(name: 'XUMLC', choices: 'tools/xumlc-7.20.0.jar', description: 'Location of the xUML Compiler')
    choice(name: 'TEST_MODE', choices: ['basic', 'regression', 'full'], description: 'Testing mode to run')
}
```
- Allows configuration of xUML compiler location
- Provides flexibility for different testing scenarios
- Enables selective test execution based on needs

## Benefits
- **Automated Code Generation**: Consistent code from UML models
- **Continuous Integration**: Early error detection
- **Automated Testing**: Comprehensive test coverage with multiple modes
- **Model-Driven Development**: UML model as single source of truth
- **Quality Assurance**: Validates model correctness and functionality
- **Artifact Management**: Preserves and tracks generated files and test results
- **Team Collaboration**: Easy sharing of compiled artifacts and test reports
- **Flexible Testing**: Multiple testing modes for different scenarios

## CI/CD Maturity Levels

| Level | Status | Description |
|-------|--------|-------------|
| **Level 1** | ✅ **Achieved** | Manual builds |
| **Level 2** | ✅ **Achieved** | Automated CI |
| **Level 3** | ✅ **Achieved** | Automated testing with multiple modes |
| **Level 4** | 🔄 **Next** | Automated deployment |
| **Level 5** | ❌ **Future** | Full CI/CD pipeline |

## Future Enhancements

### Optional Deployment Stage
When Bridge is available, add deployment:
```groovy
stage('Deploy') {
    steps {
        bat '''
            e2ebridge deploy build/urlUrl.rep -h <Bridge host> -u <user> -P <password> -o overwrite
        '''
    }
}
```

### Advanced Testing Integration
When RegTestRunner is available, enhance testing:
```groovy
stage('Advanced Tests') {
    steps {
        bat '''
            java -jar RegTestRunner.jar -project URL -suite "Tests" -logfile result.xml
        '''
    }
    post {
        always {
            junit 'result.xml'
        }
    }
}
```

## Troubleshooting
- Ensure `urlUrl.xml` exists in `URL/uml/` directory
- Verify xUML compiler JAR is in `tools/` directory
- Check Java runtime is available on Jenkins server
- Verify artifact archiving permissions in Jenkins
- Ensure test cases exist in `URL/testcase/` directories
- Check regression test suite configuration

## Expected Output
- Generated Java files in `build/` directory
- Archived artifacts available for download
- Test results and summaries in `test-results/` directory
- Compilation and test logs in Jenkins console
- Pipeline status: SUCCESS/FAILURE with detailed test reporting
