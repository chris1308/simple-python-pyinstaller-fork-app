pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                    args '-u root:root'  // Ensure we can install dependencies if needed
                }
            }
            steps {
                script {
                    // Change directory to where the sources are located
                    dir('sources') {
                        // Install dependencies like pytest if not already installed
                        sh 'pip install --upgrade pip'  // Upgrade pip
                        sh 'pip install pytest'          // Install pytest if needed
                        sh 'python -m py_compile add2vals.py calc.py'  // Compile the Python scripts
                    }
                }
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                script {
                    // Change directory to where the test files are located
                    dir('sources') {
                        sh 'mkdir -p test-reports'
                        // Run the tests using pytest
                        sh 'pytest --verbose --junit-xml test-reports/results.xml test_calc.py'
                    }
                }
            }
            post {
                always {
                    junit 'test-reports/results.xml'  // Publish the test results
                }
            }
        }
    }
}