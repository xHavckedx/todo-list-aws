pipeline {
    agent any

    environment {
	BASE_URL = "Null"
        TOKEN = credentials('token-francesc-github')
    }
    stages{
        stage('Get Code'){
            steps{
                //limpiar el workspace
                cleanWs()
                // Obtener el código del repo
                git branch: 'develop', url: 'https://github.com/xHavckedx/todo-list-aws'
            }
        }
        stage('Static Test'){
            steps{
              sh '''
                flake8 --exit-zero --format=pylint src >flake8.out
                bandit --exit-zero -r ./src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
		'''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        stage('Deploy'){
            steps{
	      sh '''
		sam deploy --config-file samconfig.toml --config-env staging //Añadir sam deploy sin el modo guided y con el config-env en staging
		'''
		def base_url = sh(script: 'aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query "Stacks[0].Outputs[?OutputKey==\'BaseUrlApi\'].OutputValue" --output text', returnStdout: true).trim()
		env.BASE_URL = base_url
            }
        }
        stage('Rest Test'){
            steps{
	      sh '''
		pytest --junitxml=result-unit.xml test/integration/todoApiTest.py 
		'''
            }
        }
        stage('Promote'){
            steps{
	      sh '''
		echo "Release: ${env.BUILD_NUMBER}" >> README.md"
		git pull --rebase
		git commit -am "BREAKING CHANGE: New release"
		git push https://JENKINS:${TOKEN}@github.com/xHavckedx/todo-list-aws.git
		'''
            }
        }
    post {
        always {
	    //Añadir todos los informes a jenkins
            junit 'result*.xml'
            //limpiar el workspace
            cleanWs()
        }
    }
  }
}
