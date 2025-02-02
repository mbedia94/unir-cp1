pipeline {
    agent none

    stages {
        stage('GetCode') {
            agent { label 'main-agent' }
            steps {
                sh '''
                    whoami
                    hostname
                    echo ${WORKSPACE}
                '''
                git 'https://github.com/mbedia94/unir-cp1.git'
            }
        }

        stage('Setup') {
            agent { label 'main-agent' }
            steps {
                sh '''
                    whoami
                    hostname
                    echo ${WORKSPACE}
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
                    agent { label 'test-agent' }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                                whoami
                                hostname
                                echo ${WORKSPACE}
                                . unir/bin/activate
                                PYTHONPATH=$(pwd) pytest --junitxml=result-unit.xml test/unit
                            '''
                            junit 'result*.xml'
                        }
                    }
                }

                stage('Coverage') {
                    agent { label 'test-agent' }
                    steps {
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            . unir/bin/activate
                            coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                            coverage xml
                        '''
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            cobertura coberturaReportFile: 'coverage.xml',
                                conditionalCoverageTargets: '100, 80, 90',
                                lineCoverageTargets: '100, 85, 95'
                        }
                    }
                }

                stage('Static') {
                    agent { label 'test-agent' }
                    steps {
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            . unir/bin/activate
                            flake8 --exit-zero --format=pylint app > flake8.out
                        '''
                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                            qualityGates: [
                                [threshold: 8, type: 'TOTAL', unstable: true], 
                                [threshold: 10, type: 'TOTAL', unstable: false]
                            ]
                    }
                }

                stage('Security') {
                    agent { label 'test-agent' }
                    steps {
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            . unir/bin/activate
                            bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                        recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                            qualityGates: [
                                [threshold: 1000, type: 'TOTAL', unstable: true], 
                                [threshold: 1500, type: 'TOTAL', unstable: false]
                            ]
                    }
                }
            }
        }

        stage('Performance') {
            agent { label 'perf-agent' }
            steps {
                sh '''
                    whoami
                    hostname
                    echo ${WORKSPACE}
                    jmeter -n -t unir.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}
