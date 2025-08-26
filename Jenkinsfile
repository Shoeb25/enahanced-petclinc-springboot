pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/practice-bala/enahanced-petclinc-springboot.git'
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
                        -Dsonar.organization=bkrrajmali \
                        -Dsonar.projectName=SpringBootPet \
                        -Dsonar.projectKey=bkrrajmali_springbootpet \
                        -Dsonar.java.binaries=.
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
        stage('Sonar Quality Gate'){
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
                }
            }
        }
    }
}
