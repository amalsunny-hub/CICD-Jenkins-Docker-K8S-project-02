pipeline{
    agent any
    environment{
        IMAGE_NAME='amalsunny27/ci-cd-k8s-project-02'
        tag="${GIT_COMMIT.substring(0,7)}"
    }
    stages{
        stage ('Checkout Code'){
            steps{
                git branch: 'main', url: 'https://github.com/amalsunny-hub/CICD-Jenkins-Docker-K8S-project-02.git'
            }

        }
        stage('Install Dependencies'){
            steps{
                bat 'pip install -r requirements.txt'
                //Because Jenkins needs the dependencies to execute unit tests during CI.The Dockerfile installs dependencies for runtime inside Kubernetes.These are two separate environments.
            }
        }
        stage('Run Unit Tests'){
            steps{
                bat 'pytest test_app.py'
            }
        }
        stage('Build Docker Image'){
            steps{
                bat  'docker build -t $IMAGE_NAME:$tag .'
            }
        }
        stage('Push Docker Image to Docker Hub'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]){
                    bat '''
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                        docker push $IMAGE_NAME:$tag
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                bat 'kubectl apply -f k8s/'
            }
        }
    }
}