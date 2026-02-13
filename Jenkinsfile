pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }

    options {
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        stage("Code pull") {
            steps {
                checkout scm
            }
        }

        stage('Build environment') {
            steps {
                echo "Creating virtual environment"
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    if [ -f requirements.txt ]; then
                        pip install -r requirements.txt
                    fi
                    mkdir -p reports
                '''
            }
        }

        stage('Static code metrics') {
            steps {
                sh '''
                    . venv/bin/activate
                    radon raw --json irisvmpy > raw_report.json || true
                    radon cc --json irisvmpy > cc_report.json || true
                    radon mi --json irisvmpy > mi_report.json || true
                    sloccount --duplicates --wide irisvmpy > sloccount.sc || true

                    coverage run irisvmpy/iris.py 1 1 2 3 || true
                    coverage xml -o reports/coverage.xml || true

                    pylint irisvmpy || true
                '''
            }
            post {
                always {
                    step([$class: 'CoberturaPublisher',
                          coberturaReportFile: 'reports/coverage.xml',
                          failNoReports: false])
                }
            }
        }

        stage('Unit tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --verbose --junit-xml=reports/unit_tests.xml || true
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'reports/unit_tests.xml'
                }
            }
        }

        stage('Acceptance tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    behave -f json -o reports/acceptance.json || true
                '''
            }
            post {
                always {
                    cucumber buildStatus: 'SUCCESS',
                             fileIncludePattern: '**/*.json',
                             jsonReportDirectory: './reports/'
                }
            }
        }

        stage('Build package') {
            steps {
                sh '''
                    . venv/bin/activate
                    python setup.py bdist_wheel
                '''
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true,
                                     artifacts: 'dist/*.whl',
                                     fingerprint: true
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
        success {
            echo "Build successful"
        }
        failure {
            echo "Build failed"
        }
    }
}
