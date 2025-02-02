pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/mbedia94/unir-cp1.git'
            }
        }

        // stage('Setup') {
        //     steps {
        //         sh '''
        //             python3 -m venv unir
        //             source unir/bin/activate.fish
        //             pip install --upgrade pip
        //             pip install -r requirements.txt
        //         '''
        //     }
        // }

        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                source unir/bin/activate.fish
                                PYTHONPATH=$(pwd) pytest --junitxml=result-unit.xml test/unit
                            '''
                            junit 'result*.xml'
                        }
                    }
                }

                stage('Coverage') {
                    steps {
                        sh '''
                            source unir/bin/activate.fish
                            coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                            coverage xml
                        '''
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100, 0, 85', lineCoverageTargets: '100, 0, 90'
                        }
                    }
                }

                stage('Static') {
                    steps {
                        sh '''
                            source unir/bin/activate.fish
                            flake8 --format=pylint app > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                            qualityGates: [
                                [threshold: 10, type: 'TOTAL', unstable: true], 
                                [threshold: 11, type: 'TOTAL', unstable: true]
                            ]
                    }
                }
            }
        }

        stage('Results') {
            steps {
                junit 'result*.xml'
                echo 'Resultados de los tests grabados correctamente.'
            }
        }
    }
}
