# DEVOPS-Docker

Docker is an open-source platform that allows you to automate the deployment, scaling, and management of applications using containerization. Containers provide a lightweight and portable way to package applications and their dependencies, enabling them to run consistently across different environments.

To build, tag, push, and run a Docker image, you can use the following Docker commands:

1. Build an Image
docker build -t <image_name> <path_to_dockerfile>
This command builds a Docker image using the Dockerfile located at <path_to_dockerfile>. The -t flag is used to provide a name or tag to the image.

2. Tag an Image
docker tag <image_name> <new_tag>
This command assigns a new tag to an existing Docker image. The <image_name> is the name or ID of the image you want to tag, and <new_tag> is the new tag you want to assign.

3. Push an Image to a Docker Registry
docker push <image_name>
This command pushes a Docker image to a Docker registry, making it available for others to download and use. The <image_name> is the name or tag of the image you want to push.

Note: Before pushing an image, you may need to log in to the Docker registry using the docker login command and providing your credentials.

4. Run a Docker Image
docker run <image_name>
This command runs a Docker image as a container. The <image_name> is the name or tag of the image you want to run. Additional flags and options can be used to customize the container's behavior, such as specifying ports, volumes, environment variables, etc.

For example, to run a container and map a local port to a container port, you can use the -p flag:

docker run -p <host_port>:<container_port> <image_name>
These commands assume that you have Docker installed and configured on your system. They are meant to provide a general overview, and there are many more advanced options and configurations available for Docker. I recommend consulting the official Docker documentation for more details and examples specific to your use case.


Jenkins Declarative Pipeline script:
pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaiswaladi246/Shopping-Cart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.url=http://13.233.102.184:9000/ \
                    -Dsonar.login=squ_815b4e28b618be7ab62693da256718391e4046d3 \
                    -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart hanish1997/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker push hanish1997/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker run -d --name shopping -p 8070:8070 hanish1997/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
