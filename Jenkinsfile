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
                    try {
                        sh '''
                            sam deploy --config-file samconfig.toml --config-env production --no-confirm-changeset
                        '''
                        def BASE_URL= sh(script: 'aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[?OutputKey==\'BaseUrlApi\'].OutputValue" --output text', returnStdout: true).trim()
                        echo "Base URL: ${BASE_URL}"
                        env.BASE_URL = BASE_URL
                    } catch (Exception e) {
                        echo "Error in Deploy stage: ${e.getMessage()}"
                        def BASE_URL= sh(script: 'aws cloudformation describe-stacks --stack-name todo-list-aws --query "Stacks[0].Outputs[?OutputKey==\'BaseUrlApi\'].OutputValue" --output text', returnStdout: true).trim()
                        echo "Base URL: ${BASE_URL}"
                        env.BASE_URL = BASE_URL
                    }
                }
            }
        }
        stage('Rest Test') {
            steps {
		script {
                    sh '''
                        export BASE_URL=${BASE_URL}
                        python -m pytest -m production --junitxml=result-unit.xml test/integration/todoApiTest.py
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

