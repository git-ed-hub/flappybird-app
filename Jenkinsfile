pipeline {
    agent any
    environment {
	    APP_NAME = "nginx-game-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "testsysadmin8"
        DOCKER_PASS = credentials("dockerhub")
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/git-ed-hub/flappybird-app.git'
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """
                    docker.withRegistry('https://index.docker.io/v1/', ${DOCKER_PASS}) {
                        docker.image(${IMAGE_NAME}).push(${IMAGE_TAG})
                        docker.image(${IMAGE_NAME}).push(${'latest'})
                    }
                }
            }
       }
       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image testsysadmin8/nginx-game-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh ('curl -v -k --user admin:${JENKINS_API_TOKEN} -H "cache-control: no-cache" -H "content-type: application/x-www-form-urlencoded" --data "IMAGE_TAG=${IMAGE_TAG}" "http://${JENKINS_SERVER}:8080//job/flappybird-deployment/buildWithParameters?token=argocd-token"')
                }
            }
       }
    }
}
