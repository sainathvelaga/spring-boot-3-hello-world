pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        def appVersion = '' //variable declaration
        APP_NAME = "my-springboot-app"
        IMAGE_TAG = "latest"
        IMAGE_NAME = "${APP_NAME}:${IMAGE_TAG}"
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_CREDENTIALS')

    }
    stages {
        stage('read the version'){
            steps{
                script{
                    readxml = readMavenPom file: ''; // Read the pom.xml file
                    // def packageJson = readJSON file: 'package.json'
                    appVersion = readxml.version
                    echo "application version: $appVersion"
                }
            }
        }
        
        stage('Build'){
            steps{
                sh """
                    mvn clean package
                """
            }
        }

        stage ('Archive artifacts'){
            steps{
                archiveArtifacts artifacts: "target/spring-boot-3-hello-world-${appVersion}.jar", fingerprint: true
                sh 'ls -la'
            }
        }


        stage('Docker build'){
            steps {
                script {
                    // Load Docker Hub credentials from Jenkins credentials store
                    withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', usernameVariable: 'DOCKERHUB_USERNAME', 
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) 
                    {
                        // Login to Docker Hub
                        sh 'echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin'
                        // Build Docker image
                        sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME} ."
                        // Tag the Docker image
                        // sh "docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} index.docker.io/${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        // Push Docker image to Docker Hub
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Deploy'){
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'springboot-helloworld-deploy', parameters: params, wait: false
                }
            }
        } 
    }
        


    
    
    post { 
        always { 
            echo 'I will always say Hello again!'
            // deleteDir()
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}