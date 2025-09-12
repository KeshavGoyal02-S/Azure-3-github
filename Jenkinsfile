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
        stage('Run PMD Analysis') {
            steps {
                 // Using a Java tool from Jenkins global tool configuration
                tool 'JDK 11'
                sh '''
                    curl -L -o pmd.zip https://github.com/pmd/pmd/releases/download/pmd_releases%2F6.55.0/pmd-bin-6.55.0.zip
                    unzip -q pmd.zip
                '''
                sh '''
                    mkdir -p pmd-reports
                    ./pmd-bin-6.55.0/bin/run.sh pmd \\
                      -d force-app/main/default/classes \\
                      -R apex-ruleset.xml \\
                      -f html \\
                      -r pmd-reports/pmd-report.html
                '''
                // Publish artifacts
                archiveArtifacts artifacts: 'pmd-reports/**'
            }
        }
        stage('Deploy Custom Object') {
            steps {
                echo 'This stage would deploy the custom object.'
                sh 'mkdir -p toDeploy'
                sh 'sfdx force:source:convert -d toDeploy -x manifest/package.xml'
                sh '''
                    sfdx force:mdapi:deploy \\
                      -u ''' + SF_USERNAME + ''' \\
                      -d ./toDeploy \\
                      -l NoTestRun \\
                      -w 10
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
