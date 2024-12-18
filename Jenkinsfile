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
                echo "Nada de nada a compilar"
                sh 'ls -la'
            }
        }
        
        stage("WireMock") {
            steps {
                sh '''
                    java -jar $(pwd)/wiremock-standalone-3.10.0.jar --port 9090 --root-dir $(pwd)/test/wiremock &
                '''
            }
        }
        
        stage("Tests"){
            parallel{
                stage("Unit"){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            sh 'PYTHONPATH=$(pwd) pytest --junitxml=result-unit.xml test/unit'
                        }
                    }
                }
        
                stage("Rest"){
                    steps{
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            sh '''
                                set -x FLASK_APP app/api.py
                                flask run &
                                PYTHONPATH=$(pwd) pytest --junitxml=result-rest.xml test/rest 
                            '''
                        }
                    }
                }
            }
        }
        
        stage("Results"){
            steps{
                junit 'result*.xml'
            }
        }
    }
}