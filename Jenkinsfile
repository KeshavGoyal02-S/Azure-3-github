pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Poll SCM every 5 minutes (adjust as needed)
    }

    environment {
        SF_CLIENT_ID = '3MVG9VMBZCsTL9hnDMtp.tRPhwcKMverfnU3lvsmfDGLo6FqFg3NnLgrtrAmJBeJHcyrIU7spU4yzBW0ifbLQ'
        SF_USERNAME = 'agentf@agentforce.com'
        SF_INSTANCE_URL = 'https://login.salesforce.com'
        PATH = "/Users/k.goyal/.nvm/versions/node/v22.17.1/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def branchName = env.GIT_BRANCH
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
                sh '''
                  curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
                  sudo apt-get install -y nodejs
                '''
            }
        }

        stage('Install Salesforce CLI') {
            steps {
                sh 'npm install -g sfdx-cli'
            }
        }

        stage('Debug JWT Key File') {
            steps {
                sh '''
                  echo "***************************************"
                  cat ./buildfiles/server.key
                  echo "***************************************"
                '''
            }
        }

        stage('Authorize Salesforce Org') {
            steps {
                sh """
                sfdx force:auth:jwt:grant \\
                    --clientid ${env.SF_CLIENT_ID} \\
                    --jwtkeyfile ./buildfiles/server.key \\
                    --username ${env.SF_USERNAME} \\
                    --instanceurl ${env.SF_INSTANCE_URL}
                """
            }
        }

        stage('Install Java 11') {
            steps {
                sh '''
                sudo apt-get update
                sudo apt-get install -y openjdk-11-jdk
                java -version
                '''
            }
        }

        stage('Download and Extract PMD') {
            steps {
                sh '''
                curl -L -o pmd.zip https://github.com/pmd/pmd/releases/download/pmd_releases%2F6.55.0/pmd-bin-6.55.0.zip
                unzip -q pmd.zip
                '''
            }
        }

        stage('Run PMD on Apex Code') {
            steps {
                sh '''
                mkdir -p pmd-reports
                ./pmd-bin-6.55.0/bin/run.sh pmd \
                  -d force-app/main/default/classes \
                  -R apex-ruleset.xml \
                  -f html \
                  -r pmd-reports/pmd-report.html
                '''
            }
        }

        stage('Publish PMD Report') {
            steps {
                archiveArtifacts artifacts: 'pmd-reports/**', fingerprint: true
            }
        }

        stage('Convert Custom Object to Metadata Format') {
            steps {
                sh '''
                mkdir -p toDeploy
                sfdx force:source:convert -d toDeploy -x manifest/package.xml
                '''
            }
        }

        stage('Deploy Custom Object Without Tests') {
            steps {
                sh """
                sfdx force:mdapi:deploy \\
                  -u ${env.SF_USERNAME} \\
                  -d ./toDeploy \\
                  -l NoTestRun \\
                  -w 10
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
