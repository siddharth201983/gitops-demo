pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "siddharth20"
        APP_NAME = "gitops-demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage("Chekout SCM"){
            steps{
                script{
                   git credentialsId: 'gitcred', 
                   url: 'https://github.com/siddharth201983/gitops-demo.git', 
                   branch: 'master'
                }
            }
        }
        stage("Build Docker Image"){
            steps{
                script{
                   docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage("Push Image to Docker"){
            steps{
                script{
                   docker.withRegistry('', REGISTRY_CREDS){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push("latest")
                   }
                }
            }
        }
        stage("Delete Docker Images"){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("Trigger gitops config pipeline"){
            steps{
                script{
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://100.25.137.130:8080/job/gitops-cd/buildWithParameters?token=gitops-config'"
                }
            }
        }
    }
}