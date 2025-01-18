
node {
    // Define environment for the whole pipeline
    def testReportDir = 'sources/test-reports'
    def deploymentDir = '/home/jenkins/python-app'
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

        stage('Manual Approval') {
            // Wait for user input to proceed or abort
            def userInput = input(
                id: 'Approval', 
                message: 'Lanjutkan ke tahap Deploy?', 
                parameters: [
                    choice(name: 'Decision', choices: ['Proceed', 'Abort'], description: 'Select an option:')
                ]
            )
            if (userInput == 'Abort') {
                error('Pipeline aborted by user.')
            }
        }

        stage('Deploy') {
            dir('sources') {
                // Ensure the deployment directory exists
                sh "mkdir -p ${deploymentDir}"

                // Copy files to the deployment directory
                sh "cp -r * ${deploymentDir}/"

                // Start the application in the background
                sh """
                cd ${deploymentDir}
                nohup python add2vals.py 5 10 > app.log 2>&1 &
                echo \$! > app.pid
                echo "Application started and is running in the background."
                """

                // Delay for 1 minute
                echo "Waiting for 1 minute while the application runs..."
                sleep(time: 1, unit: 'MINUTES')

                // Stop the application gracefully
                sh """
                cd ${deploymentDir}
                if [ -f app.pid ]; then
                    echo "Stopping the application..."
                    kill \$(cat app.pid) || echo "Application already stopped"
                    rm -f app.pid
                else
                    echo "Application PID file not found, assuming it has already stopped."
                fi
                """
            }
        }
    } finally {
        // Publish the test results using the junit plugin
        junit '**/sources/test-reports/results.xml'  // Publish the test results
    }
}
