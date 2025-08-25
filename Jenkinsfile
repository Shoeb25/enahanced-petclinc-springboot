pipeline {
    agent any
    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/practice-bala/enahanced-petclinc-springboot.git'
            }
        }       
    }
}
