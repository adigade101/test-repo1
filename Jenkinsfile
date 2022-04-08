pipeline { 
    environment { 
        registry = "adigade101/test-repo1" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
    }
    agent any 
    stages { 
        stage('Cloning our Git') { 
            steps { 
                git branch:'master', url: 'https://github.com/adigade101/test-repo1.git' 
            }
        } 
        stage('SCA using flake8') {
            steps {
                echo 'Scanning the Source Code using flake8'
                sh 'pip install flake8'
                sh 'flake8 . --format=json --output-file test.json --exit-zero'
            }
        }
        stage('Building our image') { 
            steps { 
                script { 
                    dockerImage = docker.build registry + ":$BUILD_NUMBER" 
                }
            }
        } 
        stage('Deploy our image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                } 
            }
        } 
        stage('Cleaning up') { 
            steps { 
                sh "docker rmi $registry:$BUILD_NUMBER" 
            }
        } 
        stage('Deploy to Kubernetes Dev Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                //sh "sed -i 's/BUILDNUMBER/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "sed -i 's/DEPLOYMENTENVIRONMENT/development/g' python-app.yaml"
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-app.yaml"
                sh "kubectl apply -f python-app.yaml"
            }
        }
        stage('Promote to Production') {
            steps {
                echo "Promote to production"
            }
            input {
                message "Do you want to Promote the Build to Production"
                ok "Ok"
                submitter "adityagade1992@gmail.com"
                submitterParameter "whoIsSubmitter"
                
            }
        }
        stage('Deploy to Kubernetes Production Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                sh "sed -i 's/development/production/g' python-app.yaml"
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-app.yaml"
                sh "kubectl apply -f python-app.yaml"
            }
        }

    }
}