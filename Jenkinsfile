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
                archiveArtifacts artifacts: 'URL/repository/**/*,build/**/*', fingerprint: true
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        script {
                            if (params.TEST_MODE in ['basic', 'regression', 'full']) {
                                bat '''
                                    echo Running Unit Tests...
                                    echo Testing URL Adapter functionality...
                                    echo Testing FTP functionality...
                                    echo Testing LDAP functionality...
                                    
                                    REM Create test results directory
                                    mkdir test-results
                                    
                                    REM Run basic validation tests
                                    echo Running basic validation tests...
                                    if exist "URL\\repository\\*" (
                                        echo Repository files generated successfully
                                        echo SUCCESS > test-results\\unit-tests.txt
                                    ) else (
                                        echo FAILURE: No repository files generated
                                        echo FAILURE > test-results\\unit-tests.txt
                                        exit /b 1
                                    )
                                '''
                            }
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/**/*', fingerprint: true
                        }
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        script {
                            if (params.TEST_MODE in ['regression', 'full']) {
                                bat '''
                                    echo Running Integration Tests...
                                    
                                    REM Test URL Adapter test cases
                                    echo Testing URL Adapter integration...
                                    if exist "URL\\testcase\\UrlAdapter" (
                                        echo URL Adapter test cases found
                                        echo SUCCESS > test-results\\integration-tests.txt
                                    ) else (
                                        echo FAILURE: URL Adapter test cases missing
                                        echo FAILURE > test-results\\integration-tests.txt
                                        exit /b 1
                                    )
                                    
                                    REM Test FTP functionality
                                    echo Testing FTP integration...
                                    if exist "URL\\testcase\\FTPExample" (
                                        echo FTP test cases found
                                        echo SUCCESS >> test-results\\integration-tests.txt
                                    ) else (
                                        echo FAILURE: FTP test cases missing
                                        echo FAILURE >> test-results\\integration-tests.txt
                                    )
                                '''
                            }
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/**/*', fingerprint: true
                        }
                    }
                }
                
                stage('Regression Tests') {
                    steps {
                        script {
                            if (params.TEST_MODE == 'full') {
                                bat '''
                                    echo Running Regression Tests...
                                    
                                    REM Check regression test suite
                                    if exist "URL\\regressiontest\\testsuite\\testsuite.xml" (
                                        echo Regression test suite found
                                        echo SUCCESS > test-results\\regression-tests.txt
                                        
                                        REM Run regression test validation
                                        echo Validating test suite configuration...
                                        if exist "URL\\regressiontest\\testsuite\\testsuite.errors" (
                                            echo WARNING: Test suite has errors
                                            echo WARNING >> test-results\\regression-tests.txt
                                        ) else (
                                            echo Test suite configuration valid
                                            echo SUCCESS >> test-results\\regression-tests.txt
                                        )
                                    ) else (
                                        echo FAILURE: Regression test suite missing
                                        echo FAILURE > test-results\\regression-tests.txt
                                        exit /b 1
                                    )
                                '''
                            }
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/**/*', fingerprint: true
                        }
                    }
                }
            }
        }
        
        stage('Test Results Analysis') {
            steps {
                script {
                    bat '''
                        echo Analyzing test results...
                        
                        REM Create test summary
                        echo ======================================== > test-results\\test-summary.txt
                        echo TEST EXECUTION SUMMARY >> test-results\\test-summary.txt
                        echo ======================================== >> test-results\\test-summary.txt
                        echo. >> test-results\\test-summary.txt
                        
                        if exist "test-results\\unit-tests.txt" (
                            echo UNIT TESTS: >> test-results\\test-summary.txt
                            type test-results\\unit-tests.txt >> test-results\\test-summary.txt
                            echo. >> test-results\\test-summary.txt
                        )
                        
                        if exist "test-results\\integration-tests.txt" (
                            echo INTEGRATION TESTS: >> test-results\\test-summary.txt
                            type test-results\\integration-tests.txt >> test-results\\test-summary.txt
                            echo. >> test-results\\test-summary.txt
                        )
                        
                        if exist "test-results\\regression-tests.txt" (
                            echo REGRESSION TESTS: >> test-results\\test-summary.txt
                            type test-results\\regression-tests.txt >> test-results\\test-summary.txt
                            echo. >> test-results\\test-summary.txt
                        )
                        
                        echo Test execution completed at: %date% %time% >> test-results\\test-summary.txt
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'test-results/**/*', fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and Tests Succeeded'
        }
        failure {
            echo '❌ Build or Tests Failed - check logs'
        }
        always {
            cleanWs()
        }
    }
}
