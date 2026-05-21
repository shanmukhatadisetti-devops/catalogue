pipeline{
    agent {
        label 'agent-1'
    }
    environment{
        REGION = "us-east-1"
        ACC_ID = "430774481266"
        PROJECT= "roboshop"
        COMPONENT = "catalogue"
        IMAGE_TAG = ""
        PR_NUMBER = "${env.CHANGE_ID}"
        NAMESPACE = "pr-${env.CHANGE_ID}"
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    stages{
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Debug Git') {
            steps {
                sh '''
                echo "GIT_COMMIT=$GIT_COMMIT"
                git log -1 --oneline || true
                '''
            }
        }
        stage('Set Variables') {
            steps {
                script {
                    env.IMAGE_TAG = env.GIT_COMMIT.take(7)
                    echo "IMAGE_TAG = ${env.IMAGE_TAG}"
                
        
                }
            }
        }
        stage('Install Dependencies'){
             when {
                 expression { env.CHANGE_ID == null }
            }
            steps{
                script{
                    sh"""
                        npm install
                    """
                }
            }
        }
        stage('Docker Build'){
            when {
                 expression { env.CHANGE_ID == null }
            }
            steps{
                script{
                    withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.IMAGE_TAG} .
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.IMAGE_TAG}
                        """
                    } 

                }
            }
        }
        stage('Trigger CD') {

            when {
                expression { env.CHANGE_ID != null }
            }

            steps {

                build(
                    job: '../catalogue-cd',
                    parameters: [
                        string(name: 'PR_NUMBER', value: env.CHANGE_ID),
                        string(name:  'IMAGE_TAG', value: env.IMAGE_TAG),
                        string(name:  'NAMESPACE', value: env.NAMESPACE)
                    ],
                    wait: false,
                    propagate: false
                )
            }
        }

    }
    post{
        always{
            deleteDir()
        }
    }
}