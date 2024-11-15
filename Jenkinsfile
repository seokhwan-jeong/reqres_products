pipeline { agent any

environment {
    REGISTRY = 'user14.azurecr.io'
    IMAGE_NAME = 'product'
    AKS_CLUSTER = 'user14-aks'
    RESOURCE_GROUP = 'user14-rsrcgrp'
    AKS_NAMESPACE = 'default'
    AZURE_CREDENTIALS_ID = 'Azure-Cred'
    TENANT_ID = '29d166ad-94ec-45cb-9f65-561c038e1c7a' // Service Principal 등록 후 생성된 ID
    GIT_USER_NAME = 'seokhwan-jeong'
    GIT_USER_EMAIL = 'seokhwan2@gmail.com'
    GITHUB_CREDENTIALS_ID = 'Github-Cred'
    GITHUB_REPO = 'github.com/seokhwan-jeong/reqres_products.git'
    GITHUB_BRANCH = 'master' // 업로드할 브랜치
}

stages {
    stage('Clone Repository') {
        steps {
            checkout scm
        }
    }
     
    stage('Maven Build') {
        steps {
            withMaven(maven: 'Maven') {
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    stage('Docker Build') {
        steps {
            script {
                image = docker.build("${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_NUMBER}")
            }
        }
    }
    
    stage('Push to ACR') {
        steps {
            script {
                sh "az acr login --name ${REGISTRY.split('\\.')[0]}"
                sh "docker push ${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_NUMBER}"
         
            }
        }
    }

   
     
    stage('CleanUp Images') {
        steps {
            sh """
            docker rmi ${REGISTRY}/${IMAGE_NAME}:v$BUILD_NUMBER
            """
        }
    }
    
    stage('Update deploy.yaml') {
        steps {
            script {
                sh """
                sed -i 's|image: \"${REGISTRY}/${IMAGE_NAME}:.*\"|image: \"${REGISTRY}/${IMAGE_NAME}:v${env.BUILD_ID}\"|' kubernetes/deploy.yaml
                cat kubernetes/deploy.yaml
                """
            }
        }
    }
    
    stage('Commit and Push to GitHub') {
        steps {
            script {
                withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """
                        git config --global user.email "your-email@example.com"
                        git config --global user.name "Jenkins CI"

                        cp kubernetes/deploy.yaml repo/kubernetes/deploy.yaml
                        cd repo
                        git add kubernetes/deploy.yaml
                        git commit -m "Update deploy.yaml with build ${env.BUILD_NUMBER}"
                        git push origin ${GITHUB_BRANCH}
                    """
                }
            }
        }
    } 
}
}
