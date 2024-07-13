pipeline {
    agent {
        kubernetes {
            label 'helm-agent'
            containerTemplate {
                name 'helm-jenkins-agent'
                image 'silentsmile0/helm-jenkins-agent'
                ttyEnabled true
                command 'cat'
            }
        }
    }
    
    environment {
        GIT_CREDENTIALS_ID = 'ec5690e5-cb8b-495b-8ed5-3ee490e57b12'
        REPO_URL = 'https://github.com/bar-shemtov/helm-charts.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: env.GIT_CREDENTIALS_ID, url: env.REPO_URL
            }
        }
        
        stage('Build Helm Charts') {
            steps {
                container('helm-jenkins-agent') {
                    sh 'helm package charts/sptr-backend'
                    sh 'helm package charts/sptr-frontend'
                }
            }
        }
        
        stage('Create Index.yaml') {
            steps {
                container('helm-jenkins-agent') {
                    sh 'helm repo index --url https://bar-shemtov.github.io/helm-charts .'
                }
            }
        }
        
        stage('Push Index.yaml') {
            steps {
                script {
                    // Configure Git identity
                    gitConfigureGlobal(userEmail: 'jenkins@cluster.com', userName: 'Jenkins Automation')

                    // Add and commit changes
                    sh 'git add .'
                    sh 'git commit -m "Update Helm charts" || true'  // Continue even if no changes

                    // Push changes using GitHub Push Plugin with Username with password credentials
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "git push https://${USERNAME}:${PASSWORD}@github.com/bar-shemtov/helm-charts.git main"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
