pipeline {

    agent {
        label 'Jenkins-Agent-1'
    }

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        SCANNER_HOME = tool 'SonarQube-Scanner'
        APP_NAME = "register-app"
        RELEASE = "1.0.0"
        DOCKER_REGISTRY = "sihasaneshubham"
        DOCKER_IMAGE_NAME = "${DOCKER_REGISTRY}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {

        stage('Clean Workspace'){
            steps {
                cleanWs()
            }
        }
        stage('Git Clone'){
            steps {
                git branch: 'main', credentialsId: 'GitHub-Credentials', url: 'https://github.com/shubham-sihasane/register-app.git'
            }
        }
        stage('Build Application'){
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test Application'){
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv('Sonar-Server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=RegisterApp -Dsonar.projectKey=RegisterAppKey -Dsonar.java.binaries=.
                        echo $SCANNER_HOME
                    '''
                }
            }
        }
        stage('SonarQube Quality Gate'){
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Docker Build & Push'){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Credentials') {
                        sh """
                            docker build -t ${DOCKER_IMAGE_NAME}:${IMAGE_TAG} .
                            docker push ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image  ${DOCKER_IMAGE_NAME}:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table"
            }
        }
        stage('Clean Docker Artifact') {
            steps {
                sh """
                    docker rmi ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}
                    docker rmi ${DOCKER_IMAGE_NAME}:latest
                """
            }
        }
        stage('Trigger Release Pipeline'){
            steps {
                script {
                    sh "curl -v -k --user shubhamsihasane:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://68.183.80.165:8080/job/ReleasePipeline/buildWithParameters?token=gitops-token'"
                }
            }
        }

    }

}
