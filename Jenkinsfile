pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/mbedia94/unir-cp1.git'
            }
        }

        stage('Setup') {
            steps {
                sh '''
                    python3 -m venv unir
                    . unir/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                . unir/bin/activate
                                PYTHONPATH=$(pwd) pytest --junitxml=result-unit.xml test/unit
                            '''
                            junit 'result*.xml'
                        }
                    }
                }

                stage('Coverage') {
                    steps {
                        sh '''
                            . unir/bin/activate
                            coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                            coverage xml
                        '''
                        catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                            cobertura coberturaReportFile: 'coverage.xml',
                                conditionalCoverageTargets: '100, 0, 80',
                                lineCoverageTargets: '100, 0, 90'
                        }
                    }
                }

                stage('Static') {
                    steps {
                        sh '''
                            . unir/bin/activate
                            flake8 --exit-zero --format=pylint app > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                            qualityGates: [
                                [threshold: 10, type: 'TOTAL', unstable: true], 
                                [threshold: 11, type: 'TOTAL', unstable: false]
                            ]
                    }
                }

                stage('Security') {
                    steps {
                        sh '''
                            . unir/bin/activate
                            bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                            qualityGates: [
                                [threshold: 4, type: 'HIGH', unstable: true], 
                                [threshold: 8, type: 'HIGH', unstable: false]
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
