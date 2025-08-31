pipeline {
    agent any
    tools {
        maven 'Maven'  // Must match your Maven tool name in Jenkins
    }
    environment {
        IMAGE_NAME  = "springbootapp"
        IMAGE_TAG   = "${BUILD_NUMBER}"   // dynamic tag for each build
        ACR_NAME    = "petclinicacrshoeb"
        AKS_NAME    = "petclinic-aks"
        RG_NAME     = "petclinic-rg"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'prod', url: 'https://github.com/Shoeb25/enahanced-petclinc-springboot.git'
            }
        }

        stage('Maven Validate') {
            steps {
                sh 'mvn validate'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Sonar Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarScanner'  // Must match Jenkins tool name
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.organization=shoeb25 \
                        -Dsonar.projectName=SpringBootPet \
                        -Dsonar.projectKey=Shoeb25_enahanced-petclinc-springboot \
                        -Dsonar.java.binaries=./target
                    '''
                }
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        set -e
                        TASK_FILE=target/sonar/report-task.txt
                        if [ ! -f "$TASK_FILE" ]; then echo "ERROR: $TASK_FILE not found"; exit 1; fi
                        TASK_URL=$(grep -oP "(?<=ceTaskUrl=).*" "$TASK_FILE")
                        for i in $(seq 1 180); do
                          RESP=$(curl -s -u "$SONAR_TOKEN:" "$TASK_URL")
                          STATUS=$(echo "$RESP" | jq -r '.task.status')
                          if [ "$STATUS" = "SUCCESS" ]; then ANALYSIS_ID=$(echo "$RESP" | jq -r '.task.analysisId'); break
                          elif [ "$STATUS" = "FAILED" ]; then echo "Sonar analysis FAILED"; exit 1; fi
                          sleep 5
                        done
                        [ -n "$ANALYSIS_ID" ] || { echo "Timed out waiting for analysis"; exit 1; }
                        QG=$(curl -s -u "$SONAR_TOKEN:" \
                          "https://sonarcloud.io/api/qualitygates/project_status?analysisId=$ANALYSIS_ID" \
                          | jq -r '.projectStatus.status')
                        [ "$QG" = "OK" ] || { echo "Quality Gate FAILED: $QG"; exit 1; }
                    '''
                }
            }
        }

        stage('Docker Build & Push to ACR') {
            steps {
                script {
                    docker.withRegistry("https://${ACR_NAME}.azurecr.io", "acr-credentials") {
                        def appImage = docker.build("${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}")
                        appImage.push()
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    // Get AKS credentials
                    sh "az aks get-credentials --resource-group ${RG_NAME} --name ${AKS_NAME} --overwrite-existing"

                    // Apply Kubernetes manifests
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"

                    // Update deployment with new image tag
                    sh "kubectl set image deployment/springboot-deployment springboot-container=${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}"

                    // Wait for rollout to complete
                    sh "kubectl rollout status deployment/springboot-deployment"
                }
            }
        }
    }
}
