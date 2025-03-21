# Creating a docker image using 
## Installing Docker
  - Enter the following commands to inatall and verify installation of Docker
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker --now
docker
```
![image](https://github.com/user-attachments/assets/e0a899be-cb40-40cf-9231-e35ce45a6e4f)
![image](https://github.com/user-attachments/assets/cbfdbef3-ad47-488b-89f5-0b450c27e3af)

## Download Docker plugins in Jenkins
 - Go to Jenkins `Dashboard` -> `Manage Jenkins` -> `Available Plugins` -> Search `Docker`
 - Select these plugins and Install
    - Docker
    - Docker Commons
    - Docker Pipeline
    - docker-build-step
    - CoudBees Docker Build and Publish

![image](https://github.com/user-attachments/assets/5eef535f-2b65-44ca-9ac2-2a19548477c5)
![image](https://github.com/user-attachments/assets/3f4bda54-a3dc-4811-a546-b027d83a679f)
![image](https://github.com/user-attachments/assets/f27a51c0-f7d0-4ea2-808e-161a16d296d4)

 - In this page Check the `Restart Jenkins` after installation this will restart Jenkins

![image](https://github.com/user-attachments/assets/023e655e-e8e7-4b3b-9b74-317f9f4484f2)

## Add Jenkins to Docker group
 - Go to terminal and run these commands to add Jenkins to docker group
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo reboot
```
- This will do its thing and reboot the system 

## Setting up docker credentials
 - Go to Jenkins > `Manage Jenkins` > `Credentials` > `System` > `Global Credentials (Unrestricted)` > `Add Credentials`
 -  Fill your Docker hub `username` , `password`, and in the `id` field enter `docker-seccred`

![image](https://github.com/user-attachments/assets/c9e87ffe-6776-48ae-ab68-440eb5310626)

## Creating and building a pipeline

 - Go to Jenkins `Dashboard` > `Create a Job`

![image](https://github.com/user-attachments/assets/cd3138ce-07b3-4070-97b9-0a8d76df4a36)

 - Enter a project name 
 - Select `pipeline`
 - Click `Ok`

![image](https://github.com/user-attachments/assets/3b2b76ed-f099-4257-8669-cf269e4d10a3)

 - Go to `pipeline`
 - Paste this script below and change the credential wherever mentioned:
```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "sanjai4334/docker"          // Replace with your Docker Hub username and image name
        TAG = "latest"
        CONTAINER_NAME = "my-container"
        PORT = "3001"
    }

    stages {
        
        stage('Clone Repository') {
            steps {
                echo "Cloning GitHub repository..."
                git 'https://github.com/sanjai4334/docker.git'  // Replace with your repo URL
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'chmod +x build.sh'
                sh './build.sh'
            }
        }

                stage('Login to Docker Hub') {
            steps {
                echo "Logging into Docker Hub..." // Change the cedentialsID if you have docker credentials already added with another id other than docker-seccred
                withCredentials([usernamePassword(credentialsId: 'docker-seccred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh "docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:$TAG"
                sh "docker push $IMAGE_NAME:$TAG"
            }
        }

        stage('Deploy Docker Container') {
            steps {
                echo "Deploying Docker container..."
                sh 'chmod +x deploy.sh'
                sh './deploy.sh'
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
```
 - click `save`
 - click `build`

![image](https://github.com/user-attachments/assets/7fad77f3-d083-414c-86fb-5ec0145a4b6d)

 - Go to Docker Hub to see your image pushed there

![image](https://github.com/user-attachments/assets/e07c5427-63f9-4b5e-b4c9-091cdb8cf5c1)

