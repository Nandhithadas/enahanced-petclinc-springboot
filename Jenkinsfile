pipeline {
    agent any

    tools {
        maven 'maven' // Ensure this matches the Maven installation name in Jenkins
    }

    environment {
        ImageName = 'my-app-image'
        BUILD_TAG = "latest"
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'prod', url: 'https://github.com/Nandhithadas/enahanced-petclinc-springboot.git'
            }
        }

        // stage('Maven Validate') {
        //     steps {
        //         echo 'Validating the project...'
        //         sh 'mvn validate'
        //     }
        // }

        // stage('Maven Compile') {
        //     steps {
        //         echo 'Compiling the project...'
        //         sh 'mvn compile'
        //     }
        // }

        // stage('Maven Test') {
        //     steps {
        //         echo 'Running tests...'
        //         sh 'mvn test'
        //     }
        // }

        stage('Maven Package') {
            steps {
                echo 'Packaging the project...'
                sh 'mvn package'
            }
        }

        // stage('SonarCloud Analysis') {
        //     environment {
        //         SCANNER_HOME = tool 'sonar-scanner' // Matches tool config in Jenkins
        //     }
        //     steps {
        //         withSonarQubeEnv('sonarserver') {
        //             sh '''
        //                 $SCANNER_HOME/bin/sonar-scanner \
        //                 -Dsonar.organization=nandhithadas \
        //                 -Dsonar.projectName=Jenkins \
        //                 -Dsonar.projectKey=nandhithadas_jenkins \
        //                 -Dsonar.sources=src \
        //                 -Dsonar.java.binaries=target/classes \
        //                 -Dsonar.host.url=https://sonarcloud.io
        //             '''
        //         }
        //     }
        // }

        // stage('Publish Sonar Report') {
        //     steps {
        //         echo 'Publishing SonarCloud report...'
        //         withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
        //             sh '''
        //                 mvn clean verify sonar:sonar \
        //                 -Dsonar.projectKey=nandhithadas_jenkins\
        //                 -Dsonar.organization=nandhithadas \
        //                 -Dsonar.host.url=https://sonarcloud.io \
        //                 -Dsonar.login=$SONAR_TOKEN \
        //                 -Dsonar.qualitygate.wait=false
        //             '''
        //         }
        //     }
        // }

        // stage('Build Docker Image') {
        //     steps {
        //         echo 'Building Docker image...'
        //         sh '''
        //             docker build -t ${ImageName}:${BUILD_TAG} .
        //             docker tag ${ImageName}:${BUILD_TAG} luckyregistry.azurecr.io/${ImageName}:${BUILD_TAG}
        //         '''
        //     }
        // }

        

        stage('Login to ACR and Push Image') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'azure-token', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD'),
                    string(credentialsId: 'azure-tenant', variable: 'TENANT_ID')
                ]) {
                    script {
                        echo "Logging into Azure Container Registry..."
                        sh '''
                            az login --service-principal -u "$AZURE_USERNAME" -p "$AZURE_PASSWORD" --tenant "$TENANT_ID"
                            az acr login --name luckyregistry12
                            docker push luckyregistry12.azurecr.io/${ImageName}:${BUILD_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    sh '''
                        az aks get-credentials --resource-group jeninrg --name lucky-aks-cluster11
                        kubectl apply -f k8s/springboot-deployment.yml
                        kubectl get all
                    '''
                }
            }
        }
    }
}