pipeline {
    agent { label "dev-server"}

    environment {
        DOCKER_NETWORK = "todo-network"
        TODO_IMAGE = "todo-app"
        MONGO_IMAGE = "mongo"
    }
    
    stages {
 stage('Setup Docker Network') {
            steps {
                script {
                    // Check if the Docker network exists, and create it if not
                    def networkExists = sh(script: "docker network ls | grep ${DOCKER_NETWORK}", returnStatus: true)
                    if (networkExists != 0) {
                        sh "docker network create ${DOCKER_NETWORK}"
                    }
                }
            }
        }

        stage('Build ToDo App Docker Image') {
            steps {
                script {
                    // Build the ToDo app image
                    sh "docker build -t ${TODO_IMAGE} ."
                }
            }
        }

        stage('Deploy MongoDB') {
            steps {
                script {
                    // Check if MongoDB container is already running
                    def mongoExists = sh(script: "docker ps -a | grep mongodb", returnStatus: true)
                    if (mongoExists == 0) {
                        // Stop and remove the existing MongoDB container
                        sh "docker stop mongodb || true && docker rm mongodb || true"
                    }
                    // Run MongoDB container
                    sh """
                    docker run -d \
                    --name mongodb \
                    --network ${DOCKER_NETWORK} \
                    -v mongo-data:/data/db \
                    -p 27017:27017 \
                    ${MONGO_IMAGE}
                    """
                }
            }
        }

        stage('Deploy ToDo App') {
            steps {
                script {
                    // Check if the ToDo app container is already running
                    def appExists = sh(script: "docker ps -a | grep todo-app", returnStatus: true)
                    if (appExists == 0) {
                        // Stop and remove the existing ToDo app container
                        sh "docker stop todo-app || true && docker rm todo-app || true"
                    }
                    // Run the ToDo app container
                    sh """
                    docker run -d \
                    --name todo-app \
                    --network ${DOCKER_NETWORK} \
                    -p 8000:8000 \
                    ${TODO_IMAGE}
                    """
                }
            }
        }
    }

    
        stage("code"){
            steps{
                git url: "https://github.com/LondheShubham153/node-todo-cicd.git", branch: "master"
                echo 'bhaiyya code clone ho gaya'
            }
        }
        stage("build and test"){
            steps{
                sh "docker build -t node-app-test-new ."
                echo 'code build bhi ho gaya'
            }
        }
        stage("scan image"){
            steps{
                echo 'image scanning ho gayi'
            }
        }
        stage("push"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker tag node-app-test-new:latest ${env.dockerHubUser}/node-app-test-new:latest"
                sh "docker push ${env.dockerHubUser}/node-app-test-new:latest"
                echo 'image push ho gaya'
                }
            }
        }
        stage("deploy"){
            steps{
                sh "docker-compose down && docker-compose up -d"
                echo 'deployment ho gayi'
            }
        }
        
    }
}
post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }        
