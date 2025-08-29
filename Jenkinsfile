pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE_NAME  ="springbootapp"
        IMAGE_TAG   ="latest"
        // ACR_NAME    ="fazalacr101"
        TENANT_ID   ="8beb7f81-72a1-422a-8142-a321dc9e4702"
        // ACR_LOGIN_SERVER ="${ACR_NAME}.azurecr.io"
        // FULL_IMAGE_NAME ="${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/Shoeb25/enahanced-petclinc-springboot.git'
            }
        }  
        stage('Maven Validate') {
            steps {
                echo "This is Maven Validate Stage"
                sh 'mvn validate'
            }
        }   
        stage('Maven Compile') {
            steps {
                echo "This is Maven Compile Stage"
                sh 'mvn compile'
            }
        } 
        stage('Sonar Analysis') {
            environment {
                SCANNER_HOME = tool 'Sonar-scanner'
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
                echo "This is Maven Package Stage"
                sh 'mvn package'
            }
        }
        stage('Build + Test + Sonar (Maven)') {
            steps {
                sh 'rm -f .scannerwork/report-task.txt target/sonar/report-task.txt || true'
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        mvn -B clean \
                          org.jacoco:jacoco-maven-plugin:prepare-agent \
                          verify sonar:sonar \
                          -Dsonar.organization=shoeb25 \
                          -Dsonar.projectKey=Shoeb25_enahanced-petclinc-springboot \
                          -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        } 
        stage('Sonar Quality Gate (poll)') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        set -e

                        # Use the Maven analysis report (the one with coverage)
                        TASK_FILE=target/sonar/report-task.txt
                        if [ ! -f "$TASK_FILE" ]; then
                          echo "ERROR: $TASK_FILE not found"; ls -la target || true; exit 1
                        fi

                        TASK_URL=$(grep -oP "(?<=ceTaskUrl=).*" "$TASK_FILE")
                        echo "Polling SonarCloud task: $TASK_URL"

                        # Poll up to 15 minutes (180 * 5s)
                        for i in $(seq 1 180); do
                          RESP=$(curl -s -u "$SONAR_TOKEN:" "$TASK_URL")
                          STATUS=$(echo "$RESP" | jq -r '.task.status')
                          echo "Compute Engine status: $STATUS"
                          if [ "$STATUS" = "SUCCESS" ]; then
                            ANALYSIS_ID=$(echo "$RESP" | jq -r '.task.analysisId'); break
                          elif [ "$STATUS" = "FAILED" ]; then
                            echo "Sonar analysis FAILED"; exit 1
                          fi
                          sleep 5
                        done

                        [ -n "$ANALYSIS_ID" ] || { echo "Timed out waiting for analysis"; exit 1; }

                        QG=$(curl -s -u "$SONAR_TOKEN:" \
                          "https://sonarcloud.io/api/qualitygates/project_status?analysisId=$ANALYSIS_ID" \
                          | jq -r '.projectStatus.status')

                        echo "Quality Gate: $QG"
                        [ "$QG" = "OK" ] || { echo "Quality Gate FAILED: $QG"; exit 1; }
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    echo 'Docker Build Started'
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
    }
}
