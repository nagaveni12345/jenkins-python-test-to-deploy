pipeline {
    agent any

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
                echo "Setting up virtual environment"
                sh '''
                    if [ ! -d venv ]; then
                        python3 -m venv venv
                    fi
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    mkdir -p reports
                '''
            }
        }

        stage('Unit tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --verbose --junit-xml=reports/unit_tests.xml
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'reports/unit_tests.xml'
                }
            }
        }

        stage('Build package') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
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
