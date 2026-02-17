pipeline{
    agent any
    
    environment {
        GITHUB_TOKEN = credentials('github-token')
    }
    
    stages{
        stage('Get Code'){
            steps{
                cleanWs()

                // Obtener el código fuente desde el repositorio Git (rama develop)
                echo 'Obteniendo código fuente de rama develop'
                sh '''
                    git clone -b develop https://${GITHUB_TOKEN}@github.com/albertogg1/CP1.3-4.git .
               		ls -la
					echo $WORKSPACE'''
            }
        }
        
        stage('Static Test'){
            steps{
                echo 'Análisis estático'
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh '''
						export PYTHONPATH=$WORKSPACE
                        flake8 --exit-zero --format=pylint --max-line-length=90 src/ > flake8.out
                        bandit -r src/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''

                    // Publicar informes
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                }
            }
        }
        
        stage('Deploy'){
            steps{
                echo 'Construyendo y desplegando el proyecto SAM'
                sh '''
                    sam build
                    sam deploy --config-env staging --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
        
        stage('Rest Test'){
            steps{
                script {
                    env.BASE_URL = sh( script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text",
                        returnStdout: true
                        ).trim()
                    echo "BASE_URL: ${env.BASE_URL}"
                }
 
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    export BASE_URL="${BASE_URL}"
            
                    echo "Testing at: $BASE_URL"
                    
                    # Ejecutar tests de integración
                    pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                '''
                
                // Publicar resultados de tests
                junit 'result-rest.xml'
            }
        }
        
        stage('Promote'){
            steps{
                echo 'Tests superados: Merge a master'
                sh '''
                    git config merge.ours.driver true
                    git fetch origin
                    git checkout master
                    git pull https://${GITHUB_TOKEN}@github.com/albertogg1/CP1.3-4.git master
                    
                    # Merge desde develop
                    git merge develop -m "Auto merge desde el pipeline CI"
                    
                    # Push a master usando credenciales
                    git push https://${GITHUB_TOKEN}@github.com/albertogg1/CP1.3-4.git master
                '''
            }
        }
    }
}