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
                sh 'pwd'
                sh 'ls -la'
            }
        }
        
        stage("WireMock") {
            steps {
                sh '''
                    java -jar $(pwd)/wiremock-standalone-3.10.0.jar --port 9191 --root-dir $(pwd)/test/wiremock &
                    sleep 5
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
                                export FLASK_APP=app.api
                                flask run &
                                sleep 5
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
                echo "Resultados de los tests grabados correctamente."
            }
        }
    }
}