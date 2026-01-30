pipeline{
    agent any
    environment{
        IMAGE_NAME='amalsunny27/ci-cd-k8s-project-02'
        tag="${GIT_COMMIT.substring(0,7)}"
    }
        stage('Install Dependencies'){
            steps{
                sh 'pip install -r requirements.txt'
                //Because the Jenkins needs the dependencies to execute unit tests during CI.The Dockerfile installs dependencies for runtime inside Kubernetes.These are two separate environments.
            }
        }
        stage('Run Unit Tests'){
            steps{
                sh 'pytest test_app.py'
            }
        }
        stage('Build Docker Image'){
            steps{
                sh 'docker build -t $IMAGE_NAME:$tag .'
            }
        }
        stage('Push Docker Image to Docker Hub'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]){
                    sh '''
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                        docker push $IMAGE_NAME:$tag
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}