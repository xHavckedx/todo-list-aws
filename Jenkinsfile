pipeline {
    agent any

    environment {
        TOKEN = credentials('token-francesc-github')
    }

    stages {
        stage('Get Code') {
            steps {
                // Limpiar el workspace
                cleanWs()
                // Obtener el código del repo
                git branch: 'develop', url: 'https://github.com/xHavckedx/todo-list-aws'
            }
        }
        stage('Static Test') {
            steps {
                sh '''
                    python -m flake8 --exit-zero --format=pylint src >flake8.out
                    python -m bandit --exit-zero -r ./src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        stage('Deploy') {
            steps {
                script {
                    try {
                        sh '''
                            sam deploy --config-file samconfig.toml --config-env staging --no-confirm-changeset
                        '''
                        def BASE_URL= sh(script: 'aws cloudformation describe-stacks --stack-name todo-list-aws --query "Stacks[0].Outputs[?OutputKey==\'BaseUrlApi\'].OutputValue" --output text', returnStdout: true).trim()
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
                        python -m pytest --junitxml=result-unit.xml test/integration/todoApiTest.py
                    '''
                }
            }
        }
        stage('Promote') {
            steps {
                script {
                    sh '''
                        #!/bin/bash
                        git checkout master
                        echo "Release: 1" >> README.md
                        git add README.md
                        git commit -m "BREAKING CHANGE: New release"
                        git push "https://JENKINS:${TOKEN}@github.com/xHavckedx/todo-list-aws.git"
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

