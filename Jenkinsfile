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
                // Checkout the main branch
                git branch: 'main', credentialsId: env.GIT_CREDENTIALS_ID, 
                url: env.REPO_URL
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
                    // Initialize gh-pages branch directory
                    sh 'mkdir -p gh-pages'
                    
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
                        // Check if the branch exists locally
                        script {
                            def branchExists = sh(script: 'git show-ref --verify refs/heads/gh-pages', returnStatus: true)
                            if (branchExists != 0) {
                                // If branch does not exist locally, create it
                                sh 'git checkout -b gh-pages'
                            } else {
                                // If branch exists, checkout to it
                                sh 'git checkout gh-pages'
                            }
                        }
                        
                        // Add and commit changes
                        sh 'git add .'
                        sh 'git commit -m "Update Helm charts" || true'

                        // Push changes to gh-pages branch
                        withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS_ID, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh 'git push https://' + env.GIT_USERNAME + ':' + env.GIT_PASSWORD + '@github.com/bar-shemtov/helm-charts.git ' + env.GH_PAGES_BRANCH
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
