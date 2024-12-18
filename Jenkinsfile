pipeline{
    agent any

    stages{
        stage("GetCode"){
            steps{
                git "https://github.com/mbedia94/unir-cp1.git"
            }
        }

        stage("Build"){
            steps{
                echo "Nada a compilar"
                sh "ls -la"
            }
        }

        stage("Unit"){
            steps{
                sh "PYTHONPATH=$(pwd) pytest test/unit"
            }
        }
    }
}