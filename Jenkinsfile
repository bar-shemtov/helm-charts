pipeline {
    agent {
        kubernetes {
            // Define the Kubernetes agent configuration
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
        GH_PAGES_BRANCH = 'gh-pages'
        
        // Git configuration
        GIT_EMAIL = 'baris841@gmail.com'
        GIT_NAME = 'Bar Shem-Tov'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clone the repository with both branches
                checkout([$class: 'GitSCM',
                          branches: [[name: 'main'], [name: 'gh-pages']],
                          userRemoteConfigs: [[
                              url: env.REPO_URL,
                              credentialsId: env.GIT_CREDENTIALS_ID
                          ]]])
            }
        }
        
        stage('Configure Git') {
            steps {
                // Set Git global configuration
                withEnv(['GIT_COMMITTER_EMAIL=${env.GIT_EMAIL}', 'GIT_COMMITTER_NAME=${env.GIT_NAME}']) {
                    sh 'git config --global user.email "${env.GIT_EMAIL}"'
                    sh 'git config --global user.name "${env.GIT_NAME}"'
                }
            }
        }
        
        stage('Build Helm Charts') {
            steps {
                container('helm-jenkins-agent') {
                    // Build Helm charts here
                    sh 'helm package charts/sptr-backend'
                    sh 'helm package charts/sptr-frontend'
                }
            }
        }
        
        stage('Create and Push Index.yaml') {
            steps {
                container('helm-jenkins-agent') {
                    // Copy packaged charts to gh-pages directory
                    sh 'cp *.tgz gh-pages/'
                    
                    // Create or update the index.yaml file
                    sh 'helm repo index gh-pages --url https://bar-shemtov.github.io/helm-charts'
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                container('helm-jenkins-agent') {
                    dir('gh-pages') {
                        // Add and commit changes
                        sh 'git add .'
                        sh 'git commit -m "Update Helm charts" || true'
                        
                        // Push changes using Git plugin
                        withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            git push {
                                credentialsId(env.GIT_CREDENTIALS_ID)
                                branch(env.GH_PAGES_BRANCH)
                                url(env.REPO_URL)
                            }
                        }
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
