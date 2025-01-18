
node {
    // Define environment for the whole pipeline
    def testReportDir = 'sources/test-reports'
    def deploymentDir = '/var/www/react-app'
    try {
        stage('Build') {
            // Run the build stage in a Docker container with the image 'python:2-alpine'
            docker.image('python:2-alpine').inside('-u root:root') {
                // Ensure that the necessary files are present
                dir("sources"){
                    sh 'ls -l'  // List files to make sure add2vals.py and calc.py are present
                    // Install dependencies like pytest if not already installed
                    sh 'pip install --upgrade pip'  // Upgrade pip
                    sh 'pip install pytest'          // Install pytest if needed
                    // Compile the Python scripts
                    sh 'python -m py_compile add2vals.py calc.py'
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
        stage('Deploy') {
            // Deploy application to a local directory
            stage('Deploy to Localhost') {
                dir('sources') {
                    // Ensure the deployment directory exists
                    sh "mkdir -p ${deploymentDir}"

                    // Copy files to the deployment directory
                    sh "cp -r * ${deploymentDir}/"

                    // Run the application
                    sh "cd ${deploymentDir} && python add2vals.py 5 10 &"
                }
            }
        }
    } finally {
        // Publish the test results using the junit plugin
        junit '**/sources/test-reports/results.xml'  // Publish the test results
    }
}
