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
                echo 'Obteniendo código fuente de rama master'
                sh '''
                    git clone -b master https://${GITHUB_TOKEN}@github.com/albertogg1/CP1.3-4.git .
               		ls -la
					echo $WORKSPACE'''
            }
        }
               
        stage('Deploy'){
            steps{
                echo 'Construyendo y desplegando el proyecto SAM'
                sh '''
                    sam build
                    sam deploy --config-env production --no-confirm-changeset --no-fail-on-empty-changeset
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
                    pytest -m read --junitxml=result-rest.xml test/integration/todoApiTest.py
                '''
                
                // Publicar resultados de tests
                junit 'result-rest.xml'
            }
        }
    }
}