pipeline {
    agent any
    environment {
        SF_CONSUMER_KEY = "${env.SF_CONSUMER_KEY}"
        SF_USERNAME = "${env.SF_USERNAME}"
        SF_INSTANCE_URL = "${env.SF_INSTANCE_URL ?: 'https://login.salesforce.com'}"
    }
    tools {
        // You MUST configure a NodeJS tool in Jenkins > Manage Jenkins > Global Tool Configuration
        // with the name 'NodeJS 18.x' for this pipeline to work.
        nodejs 'NodeJS 18.x'
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    def branchName = env.GIT_BRANCH.replaceFirst('origin/', '')
                    echo "Checking branch name..."
                    if (branchName != 'main') {
                        error "This pipeline only runs on 'main' branch. Current branch: ${branchName}"
                    }
                    echo "Current branch: ${branchName}"
                }
            }
        }
        stage('Install Node.js') {
            steps {
                // The NodeJS tool is automatically installed and added to the PATH
                // by the 'tools' directive above.
                echo 'Node.js is ready to use.'
            }
        }
        stage('Install Salesforce CLI') {
            steps {
                // Assuming Salesforce CLI is a node package, we will install it globally.
                sh 'npm install sfdx-cli -g'
            }
        }
        stage('Debug JWT Key File') {
            steps {
                echo 'This stage would typically verify the presence of a JWT key file.'
                // You can add your verification script here, for example:
                // sh 'ls -l server.key'
            }
        }
        stage('Authorize Salesforce Org') {
            steps {
               // withCredentials block to handle the secure file
               withCredentials([file(credentialsId: 'SF_SERVER_KEY', variable: 'server_key_file')]) {
                    // Script block to run bash commands
                    sh '''
                        echo "***************************************"
                        cat "${server_key_file}"
                        echo "***************************************"
                    '''

                    sh '''
                        sfdx force:auth:jwt:grant \\
                          --clientid ''' + SF_CONSUMER_KEY + ''' \\
                          --jwtkeyfile ''' + server_key_file + ''' \\
                          --username ''' + SF_USERNAME + ''' \\
                          --instanceurl ''' + SF_INSTANCE_URL + '''
                    '''
                }
            }
        }
        stage('Install Java 11') {
            steps {
                echo 'This stage is a placeholder. You can either manually install Java on your agent or use a tool installer plugin if available.'
            }
        }
        stage('Download and Extract PMD') {
            steps {
                echo 'This stage would download PMD'
            }
        }
        stage('Run PMD on Apex Code') {
            steps {
                echo 'This stage would run PMD on your Apex files.'
            }
        }
        stage('Publish PMD Report') {
            steps {
                echo 'This stage would publish the PMD report.'
            }
        }
        stage('Convert Custom Object to Metadata Format') {
            steps {
                echo 'This stage would convert the custom object to the metadata format.'
                // Example using Salesforce CLI
                // sh 'sfdx force:source:convert'
            }
        }
        stage('Deploy Custom Object Without Tests') {
            steps {
                echo 'This stage would deploy the custom object.'
                // Example using Salesforce CLI
                // sh 'sfdx force:source:deploy --checkonly --sourcepath=path/to/your/source'
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
