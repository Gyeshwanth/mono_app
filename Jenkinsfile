pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "java-registration-app"
        RELEASE = "1.0.0"
        DOCKER_USER = "yeshwanthgosi"
        DOCKER_PASS = 'dockerhub-token'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	
    }
    stages {
         stage('clean workspace') {
            steps {
                cleanWs()
            }
         }
         stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Gyeshwanth/mono_app.git'
            }
         }
         stage ('Build Package')  {
	         steps {
                dir('webapp'){
                sh "mvn package"
                }
             }
         }
         stage ('SonarQube Analysis') {
            steps {
              withSonarQubeEnv('SonarQube-Server') {
                dir('webapp'){
                sh 'mvn -U clean install sonar:sonar'
                }
              }  
            }
         }
         stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
         }
       
         stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                     }
                 }
             }
         }
         stage("Trivy Image Scan") {
             steps {
                 script {
	                  sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image yeshwanthgosi/java-registration-app:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
                 }
             }
         }
         stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }

	  stage('Deploy to Kubernets'){
             steps{
                 script{
                      dir('Kubernetes') {
                         kubeconfig(credentialsId: 'kubernetes', serverUrl: '') {
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                         sh 'kubectl rollout restart deployment.apps/registerapp-deployment'
                         }   
                      }
                 }
             }
         }    

      

	    

    }

post {
      always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'yeshu.fsd@gmail.com',                              
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
      }
    }
	
}    
