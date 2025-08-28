pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME  ="springbootapp"
        IMAGE_TAG   ="latest"
        ACR_NAME    ="jenkinsazure"
        TENANT_ID   ="ec78375d-0db0-42cf-82a6-2e6403e95936"
        ACR_LOGIN_SERVER ="${ACR_NAME}.azurecr.io"
        FULL_IMAGE_NAME ="${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        RESOURCE_GROUP ="demo-rg"
        CLUSTER_NAME ="demo-eks"
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/practice-bala/enahanced-petclinc-springboot.git'
            }
        }  
        // stage('Maven Validate') {
        //     steps {
        //         echo "This is Maven Validate Stage"
        //         sh 'mvn validate'
        //     }
        // }   
        // stage('Maven Compile') {
        //     steps {
        //         echo "This is Maven Compile Stage"
        //         sh 'mvn compile'
        //     }
        // } 
        // stage('Sonar Analysis') {
        //     environment {
        //         SCANNER_HOME = tool 'Sonar-scanner'
        //     }
        //     steps {
        //         withSonarQubeEnv('sonarserver') {
        //             sh '''
        //                 $SCANNER_HOME/bin/sonar-scanner \
        //                 -Dsonar.organization=bkrrajmali \
        //                 -Dsonar.projectName=SpringBootPet \
        //                 -Dsonar.projectKey=bkrrajmali_springbootpet \
        //                 -Dsonar.java.binaries=.
        //             '''
        //         }
        //     }
        // }
         stage('Maven Package') {
            steps {
                echo "This is Maven Package Stage"
                sh 'mvn package'
            }
        } 
        // stage('Sonar Quality Gate'){
        //     steps {
        //         timeout(time: 1, unit: 'MINUTES') {
        //         waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
        //         }
        //     }
        // }
        stage ('Docker Build'){
            steps {
                script {
                    echo 'Docker Build Started'
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        stage ('ACR LOGIN'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME',passwordVariable: 'AZURE_PASSWORD')]){
                    script {
                        echo "Azure login to container registry"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        az acr login --name $ACR_NAME
                        '''
                    }
                }
            }
        }
        stage('Docker Push to ACR'){
            steps {
                script {
                    echo "Docker Push image to Registry" 
                    sh '''
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
                    docker push ${FULL_IMAGE_NAME}
                    '''
                }
            }
        }
       stage('Azure Login to AKS Cluster') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME',passwordVariable: 'AZURE_PASSWORD')]){
            script {
                echo "Azure Loginto AKS"
                sh '''
                az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
                '''
            }    
         }
       }
     }
     stage ('Deploy to AKS') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-sp', usernameVariable: 'AZURE_USERNAME',passwordVariable: 'AZURE_PASSWORD')]){
            script {
                    echo "Azure Loginto AKS"
                    sh '''
                    az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                    kubectl apply -f k8s/sprinboot-deployment.yaml
                    '''
                    }
                }
            }
        }
    }
}

