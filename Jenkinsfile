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
        stage('Maven Compile') {
            steps {
                echo "This is Maven Compile Stage"
                sh 'mvn validate'
            }
        }       
    }
}
