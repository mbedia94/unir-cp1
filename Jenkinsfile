pipeline {
    agent any

    stages {
        stage("GetCode") {
            steps {
                git "https://github.com/mbedia94/unir-cp1.git"
            }
        }

        stage("Build") {
            steps {
                echo "Nada a compilar"
                script {
                    sh 'ls -la'
                }
            }
        }

        stage("Unit") {
            steps {
                script {
                    sh 'PYTHONPATH=$(pwd) pytest test/unit'
                }
            }
        }
    }
}
