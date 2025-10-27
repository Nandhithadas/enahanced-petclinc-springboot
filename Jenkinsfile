pipeline {
    agent any

    tools {
        maven 'maven' // Ensure this matches the Maven installation name in Jenkins
    }

    environment {
        IMAGE_NAME = 'my-app-image'
        IMAGE_TAG = 'latest'
        BUILD_TAG = "latest"
        ACR_NAME = 'luckyregistry12'
        ACR_LOGIN_SERVER = 'luckyregistry12.azurecr.io'
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        TENANT_ID = "9f0886a2-d016-4cc8-8f25-ec95b841aa78"
        RG              = "jenkinrg"
        NAME            = "lucky-aks-cluster"
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/Nandhithadas/enahanced-petclinc-springboot.git'
            }
        }

        stage('Maven Validate') {
            steps {
                echo 'Validating the project...'
                sh 'mvn validate'
            }
        }

        stage('Maven Compile') {
            steps {
                echo 'Compiling the project...'
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Maven Package') {
            steps {
                echo 'Packaging the project...'
                sh 'mvn package'
            }
        }

        stage('SonarCloud Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner' // Matches tool config in Jenkins
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.organization=nandhithadas \
                        -Dsonar.projectName=Jenkins \
                        -Dsonar.projectKey=nandhithadas_jenkins \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }

        stage('Publish Sonar Report') {
            steps {
                echo 'Publishing SonarCloud report...'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=nandhithadas_jenkins\
                        -Dsonar.organization=nandhithadas \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.qualitygate.wait=false
                    '''
                }
            }
        }

       

        

         stage('Azure Login TO ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-token', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) {
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

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    sh '''
                        az aks get-credentials --resource-group jenkinrg --name lucky-aks-cluster11
                        kubectl apply -f k8s/springboot-deployment.yaml
                        kubectl get all
                    '''
                }
            }
        }
    }
}