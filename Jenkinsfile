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
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-21.0.7.0.6-1.el9.x86_64/bin/java'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
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
                    JAVA_HOME mvn clean package
                """
            }
        }

        stage ('Archive artifacts'){
            steps{
                archiveArtifacts artifacts: "spring-boot-helloworld-${appVersion}.war", fingerprint: true
                sh 'ls -la'
            }
        }


        stage('Docker build'){
            steps {
                script {
                    // Load Docker Hub credentials from Jenkins credentials store
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKERHUB_USERNAME', 
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) 
                    {
                        // Login to Docker Hub
                        sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                        // Build Docker image
                        sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                        // Tag the Docker image
                        sh "docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} index.docker.io/${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        // Push Docker image to Docker Hub
                        sh "docker push index.docker.io/${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
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