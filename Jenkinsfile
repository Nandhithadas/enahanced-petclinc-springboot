pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment{
        IMAGE_NAME = 'springbootapp'
        IMAGE_TAG = 'latest'
        TENANT_ID ='9f0886a2-d016-4cc8-8f25-ec95b841aa78'
        ACR_NAME = 'luckyregistry11'
        ACR_LOGIN_SERVER = 'luckyregistry11.azurecr.io'
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        RG              = "demo11"
        NAME            = "lucky-aks-cluster11"
        SONAR_TOKEN = credentials('sonar-token')
    }
    stages {
        stage('Checkout FROM GIT') {
            steps {
                git branch: 'prod' , url: 'https://github.com/bkrrajmali/enahanced-petclinc-springboot.git'
        }
      }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
            steps {
                withSonarQubeEnv('sonar-token') {  // matches the Name in Jenkins → Configure System → SonarQube servers
                    sh 'mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}'
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


         
        stage('Docker Build') {
            steps {
                script {
                    echo "Building Docker Image......."
                    docker.build ("${IMAGE_NAME}:${IMAGE_TAG}") 
                }
            }
        }
        stage('Azure Login TO ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) {
                    script {
                        echo "Azure Login Started"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        az acr login --name $ACR_NAME
                        '''
                    }
                }
            }
        }
        stage('Docker Push to ACR') {
            steps {
                script {
                    echo "Docker Image Push to ACR"
                    sh '''
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
                   
                    docker push ${FULL_IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Azure Login TO AKS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) {
                    script {
                        echo "Azure Login to AKS"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        az aks get-credentials --resource-group $RG --name $NAME --overwrite-existing
                        '''
                    }
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) {
                    script {
                        echo "Azure Login to AKS"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        kubectl apply -f k8s/sprinboot-deployment.yaml
                        '''
                    }
                }
            }
        }
    }
}