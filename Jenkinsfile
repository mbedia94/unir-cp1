pipeline {
    agent none

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout()
    }

    stages {
        stage('GetCode') {
            agent { label 'main-agent' }
            steps {
                checkout scm
                sh '''
                    whoami
                    hostname
                    echo ${WORKSPACE}
                '''
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
                        checkout scm
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            python3 -m venv unir
                            . unir/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
                            PYTHONPATH=$(pwd) pytest --junitxml=result-unit.xml test/unit
                        '''
                        junit 'result*.xml'
                    }
                }

                stage('Coverage') {
                    agent { label 'test-agent' }
                    steps {
                        checkout scm
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            python3 -m venv unir
                            . unir/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
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
                        checkout scm
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            python3 -m venv unir
                            . unir/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
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
                        checkout scm
                        sh '''
                            whoami
                            hostname
                            echo ${WORKSPACE}
                            python3 -m venv unir
                            . unir/bin/activate
                            pip install --upgrade pip
                            pip install -r requirements.txt
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
                checkout scm
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
