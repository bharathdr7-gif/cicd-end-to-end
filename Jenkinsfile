pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
            steps {
                git credentialsId: 'github-credentials', 
                    url: 'https://github.com/bharathdr7-gif/cicd-end-to-end',
                    branch: 'main'
            }
        }

        stage('Build Docker'){
            steps {
                script {
                    sh '''
                        echo 'Build Docker Image'
                        docker build -t bharathdr7/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USERNAME',
                            passwordVariable: 'DOCKER_PASSWORD'
                        )
                    ]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login \
                                -u "$DOCKER_USERNAME" \
                                --password-stdin

                            docker push bharathdr7/cicd-e2e:${BUILD_NUMBER}

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github-credentials', 
                    url: 'https://github.com/bharathdr7-gif/cicd-demo-manifests-repo.git',
                    branch: 'main'
            }
        }

        stage('Update K8S manifest & push to Repo'){
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-credentials',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                            cat deploy.yaml

                            sed -i "s/32/${BUILD_NUMBER}/g" deploy.yaml

                            cat deploy.yaml

                            git config user.name "Jenkins"
                            git config user.email "jenkins@localhost"

                            git add deploy.yaml
                            git commit -m "Updated deploy yaml | Jenkins Pipeline" || true

                            git remote -v

                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/bharathdr7-gif/cicd-demo-manifests-repo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
