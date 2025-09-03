pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage("Cleaning the Workspace") {
            steps {
                cleanWs()
            }
        }
        stage('Cloning The Code From Github') {
            steps {
                git url: "https://github.com/DattaRahegaonkar/Two-Tier-Flask-App.git", branch: "master"
            }
        }
        stage('Scanning the Code Using SonarQube') {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Flask-App -Dsonar.projectKey=Flask-App "
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Scanning File System Using Trivy") {
            steps {
                sh "trivy fs . --format table -o trivy-fs-report.html"
            }
        }
        stage("Building the Docker Image") {
            steps {
                sh 'docker build -t flaskapp .'
            }
        }
        stage("Docker Image Scanning Using Trivy") {
            steps {
                sh 'trivy image flaskapp --format table -o trivy-img-report.html --exit-code 0 --severity HIGH,CRITICAL || true'
            }
        }
        stage("Pushing the Image on Docker Hub Repository") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "docker_hub_creds",
                    usernameVariable: "docker_hub_user",
                    passwordVariable: "docker_hub_password"
                )]) {
                    sh "docker login -u $docker_hub_user -p $docker_hub_password"
                    sh "docker tag flaskapp $docker_hub_user/flaskapp:latest"
                    sh "docker push $docker_hub_user/flaskapp:latest"
                }
            }
        }
        stage("Deploying Using Docker Compose") {
            steps {
                sh 'docker network inspect flask-app-network >/dev/null 2>&1 || docker network create flask-app-network'
                sh "docker rm -f flaskapp || true"
                sh "docker-compose up -d"
            }
        }
        // stage('Deploy to Kind Cluster') {             // If you want to deploy on Kubernetes Cluster, uncomment this stage and comment the above "Deploying Using Docker Compose" stage.
        //     steps {
        //         sh 'kubectl apply -f k8s/namespace.yml'
        //         sh 'kubectl apply -f k8s/deployment.yml'
        //         sh 'kubectl apply -f k8s/service.yml'
        //         sh 'kubectl apply -f k8s/ingress.yml'
        //     }
        // }
    }
    post {
        always {
            archiveArtifacts artifacts: '*.html', allowEmptyArchive: true
        }
        success {
            script {
                emailext(
                    to: 'rahegaonkardatta@gmail.com',
                    subject: "✅ SUCCESS: Flask App CICD - Build #${BUILD_NUMBER}",
                    body: """
                    Hi Team,
                    
                    The Flask App CICD pipeline completed successfully 🎉

                    👉 Build Number : ${BUILD_NUMBER}  
                    👉 Build Status : ${currentBuild.currentResult}  
                    👉 Build Logs   : ${BUILD_URL}  

                    Reports attached (Trivy, OWASP).

                    Regards,  
                    Jenkins
                    """,
                    attachmentsPattern: '*.html',
                    mimeType: 'text/plain'
                )
            }
        }
        failure {
            script {
                emailext(
                    to: 'rahegaonkardatta@gmail.com',
                    subject: "❌ FAILED: Flask App CICD - Build #${BUILD_NUMBER}",
                    body: """
                    Hi Team,
                    
                    The Flask App CICD pipeline has FAILED 🚨

                    👉 Build Number : ${BUILD_NUMBER}  
                    👉 Build Status : ${currentBuild.currentResult}  
                    👉 Build Logs   : ${BUILD_URL}  

                    Please check the logs for details.  
                    Reports attached (if generated).

                    Regards,  
                    Jenkins
                    """,
                    attachmentsPattern: '*.html',
                    mimeType: 'text/plain'
                )
            }
        }
    }
}
