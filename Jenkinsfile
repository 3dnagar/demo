pipeline {
    agent any
    
    triggers {
        cron('H/2 * * * *')
    }

    tools {
        jdk 'jdk8'
        maven 'mvn'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stage', 'prod'],
            description: 'Select environment'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Selected environment: ${params.ENVIRONMENT}"
                git branch: 'master',
                    url: 'https://github.com/opstree/spring3hibernate.git'
            }
        }

        stage('Validate Environment') {
            steps {
                script {
                    if (params.ENVIRONMENT != 'prod') {
                        error "Build aborted: Only PROD builds are allowed"
                    }
                }
            }
        }

        stage('Manual Approval (Prod)') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: "Approve build for PRODUCTION?",
                          ok: "Proceed"
                }
            }
        }

        stage('GitLeaks Scan') {
            steps {
                sh '''
                gitleaks detect \
                  --source . \
                  --report-format json \
                  --report-path gitleaks-report.json \
                  --no-git || true
                '''
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Tag WAR') {
            steps {
                sh '''
                WAR=$(ls target/*.war | head -n 1)
                cp "$WAR" "target/Spring3HibernateApp-$ENVIRONMENT-$BUILD_NUMBER.war"
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '''
                    target/Spring3HibernateApp-*-*.war,
                    gitleaks-report.json
                ''',
                fingerprint: true
            }
        }
    }

    post {
        success {
            echo "✅ BUILD SUCCESSFUL"

            emailext(
                subject: "SUCCESS: Jenkins Build #${env.BUILD_NUMBER}",
                body: """
Build SUCCESSFUL ✅

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Environment: ${params.ENVIRONMENT}

Artifacts:
${env.BUILD_URL}artifact/

Console:
${env.BUILD_URL}console
""",
                to: "3dnagar@gmail.com"
            )
        }

        failure {
            echo "❌ BUILD FAILED"

            emailext(
                subject: "FAILURE: Jenkins Build #${env.BUILD_NUMBER}",
                body: """
Build FAILED ❌

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Environment: ${params.ENVIRONMENT}

Check Console Logs:
${env.BUILD_URL}console
""",
                to: "3dnagar@gmail.com"
            )
        }

        aborted {
            emailext(
                subject: "ABORTED: Jenkins Build #${env.BUILD_NUMBER}",
                body: """
Build ABORTED ⏹️

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Environment: ${params.ENVIRONMENT}
""",
                to: "3dnagar@gmail.com"
            )
        }
    }
}
