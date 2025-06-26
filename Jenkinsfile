pipeline {
    agent any

    environment {
        registryUrl = 'docker.io'
        registryCredential = 'docker-login'  // ID configured in Jenkins Credentials
        dockerImageName = 'sonalikurade/devops'
        dockerImage = ''
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/TechOpsBySonali/spring-petclinic'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                     def dockerImageName = "sonalikurade/devops:latest"
                     def dockerImage = docker.build(dockerImageName, "-f .devcontainer/Dockerfile .")

                     docker.withRegistry('https://index.docker.io/v1/', 'docker-login') {
                         dockerImage.push()
                     }
                }
            }
         
        }
        
 

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'gcp-login', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    
                        echo "Installing gke-gcloud-auth-plugin..."
                        sudo apt-get update -y
                        sudo apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin
                        
                        echo "Activating GCP service account..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS

                        echo "Getting GKE credentials..."
                        gcloud container clusters get-credentials my-cluster \
                          --zone us-west1-b \
                          --project seventh-reef-463513-t3

                        echo "Applying Kubernetes manifests..."
                        kubectl apply -f ./k8s/db.yml --validate=false
                        kubectl apply -f ./k8s/petclinic.yml --validate=false
                       
                    '''
                }
            }
        }
    }
}
