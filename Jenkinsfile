node {
    // Define environment for the whole pipeline
    def testReportDir = 'sources/test-reports'

    try {
        stage('Build') {
            // Run the build stage in a Docker container with the image 'python:2-alpine'
            docker.image('python:2-alpine').inside('-u root:root') {
                // Change directory to where the sources are located and run build steps
                dir('sources') {
                    // Install dependencies like pytest if not already installed
                    sh 'pip install --upgrade pip'  // Upgrade pip
                    sh 'pip install pytest'          // Install pytest if needed
                    sh 'python -m py_compile add2vals.py calc.py'  // Compile the Python scripts
                }
            }
        }

        stage('Test') {
            // Run the test stage in a Docker container with the image 'qnib/pytest'
            docker.image('qnib/pytest').inside() {
                // Change directory to where the test files are located
                dir('sources') {
                    // Ensure the test-reports directory exists
                    sh "mkdir -p ${testReportDir}"
                    
                    // Run the tests using pytest and generate XML report
                    sh "pytest --verbose --junit-xml=${testReportDir}/results.xml test_calc.py"
                    
                    // List files in the test-reports directory to verify the report exists
                    sh "ls -l ${testReportDir}"
                }
            }
        }
    } finally {
        // Publish the test results using the junit plugin
        junit '**/sources/test-reports/results.xml'  // Publish the test results
    }
}
