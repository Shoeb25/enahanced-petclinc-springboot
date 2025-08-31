pipeline {
    agent any
    tools {
        maven 'Maven'  // Make sure this matches your Jenkins Maven tool name
    }
    environment {
        IMAGE_NAME = "springbootapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        ACR_NAME   = "petclinicacrshoeb"
        ACR_URL    = "${ACR_NAME}.azurecr.io"
        RG_NAME    = "petclinic-rg"
        AKS_NAME   = "petclinic-aks"
        AZ_CLI     = "/usr/bin/az"  // Path to az CLI on your Jenkins node
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/Shoeb25/enahanced-petclinc-springboot.git'
            }
        }

        stage('Maven Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=Shoeb25_enahanced-petclinc-springboot'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Verify k8s Manifests') {
            steps {
                sh 'ls -la k8s/'
            }
        }

        stage('Docker Build & Push to ACR') {
            steps {
                script {
                    docker.withRegistry("https://${ACR_URL}", 'acr-credentials') {
                        def image = docker.build("${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}")
                        image.push()
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    sh "${AZ_CLI} aks get-credentials --resource-group ${RG_NAME} --name ${AKS_NAME} --overwrite-existing"
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                    sh "kubectl set image deployment/springboot-deployment springboot-container=${ACR_URL}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "kubectl rollout status deployment/springboot-deployment"
                }
            }
        }
    }
}
