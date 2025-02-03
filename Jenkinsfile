pipeline {
    agent {
        label 'Jenkins-Agent-1'
    }

    environment {
        APP_NAME = "register-app"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout'){
             steps {
                git branch: 'main', credentialsId: 'GitHub-Credentials', url: 'https://github.com/shubham-sihasane/gitops-register-app.git'
             }
        }
        stage('Update Deployment Tags'){
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }
        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "Shubham Sihasane"
                   git config --global user.email "shubhamsihasane101@gmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'GitHub-Credentials', gitToolName: 'Default')]) {
                  sh "git push https://github.com/shubham-sihasane/gitops-register-app.git main"
                }
            }
        }
    }
}
