pipeline {
    agent any

    environment {
        BASE_URL = "Null"
        TOKEN = credentials('token-francesc-github')
    }

    stages {
        stage('Get Code') {
            steps {
                // Limpiar el workspace
                cleanWs()
                // Obtener el código del repo
                git branch: 'master', url: 'https://github.com/xHavckedx/todo-list-aws'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh '''
                        sam deploy --config-file samconfig.toml --config-env production --no-confirm-changeset
                    '''
                    def base_url = sh(script: 'aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[?OutputKey==\'BaseUrlApi\'].OutputValue" --output text', returnStdout: true).trim()
                    env.BASE_URL = base_url
                }
            }
        }
        stage('Rest Test') {
            steps {
                sh '''
                    python -m pytest --junitxml=result-unit.xml -m production test/integration/todoApiTest.py
                '''
            }
        }
        stage('Promote') {
            steps {
                script {
                    sh '''
                        echo "Release: ${env.BUILD_NUMBER}" >> README.md
                        git pull --rebase
                        git commit -am "BREAKING CHANGE: New release"
                        git push https://JENKINS:${TOKEN}@github.com/xHavckedx/todo-list-aws.git
                    '''
                }
            }
        }
    }

    post {
        always {
            // Añadir todos los informes a Jenkins
            junit 'result*.xml'
            // Limpiar el workspace
            cleanWs()
        }
    }
}

