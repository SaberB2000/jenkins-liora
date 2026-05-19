pipeline {
    agent any 
    environment {
        DOCKER_ID = "sbelaidi"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    stages {
        stage('Building') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate

                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
                
            }
        }
        stage('Testing') {

            steps {
                sh '''
                    . .venv/bin/activate
                    python -m unittest
                '''

            }
            
        }
        stage('Deploying') {
            steps{
                script {
                    sh '''
                        docker rm -f jenkins
                        docker build -t ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('User acceptance') {
            steps {
                input message: "Proceed to push to main", ok: "Yes"
                
            }
        }
        stage('Pushing and Merging') {
            parallel {
                stage('Pushing') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
                    }
                    steps{
                        sh '''
                             echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                             docker push ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
                stage('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }

    }
    post {
        always {
            sh "Docker logout"
        }
    }
}