pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("code") {
            steps {
                git "https://github.com/Shiva-Bornarch/one.git"
            }
        }

        stage("CQA") {
            steps {
                withSonarQubeEnv('MYSONAR') {
                    sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=myproj -Dsonar.projectName='myproj'"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage("Build and test") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Docker images") {
            steps {
                sh "docker build -t dudishivaram/java:${IMAGE_TAG} ."
            }
        }

        stage("image scan") {
            steps {
                sh 'trivy image tomcatimage2'
            }
        }

        stage("Docker login") {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage("Registry") {
            steps {
                sh 'docker push dudishivaram/java:${IMAGE_TAG}'
            }
        }

        stage("Update Manifest Repo") {
            steps {
                sh """
                rm -rf One-ArgoCD
                git clone git@github.com:Shiva-Bornarch/One-ArgoCD.git
                cd One-ArgoCD/k8s-manifests

                sed -i 's|image: .*|image: dudishivaram/java:${IMAGE_TAG}|' deployment.yaml

                git config user.email "shivaram@gmail.com"
                git config user.name "shivaram"

                git commit -am "Update image to ${IMAGE_TAG}"
                git push origin main
                """
            }
        }
    }
}


// cp /root/.ssh/id_rsa /var/lib/jenkins/.ssh/
// cp /root/.ssh/id_rsa.pub /var/lib/jenkins/.ssh/
// cp /root/.ssh/known_hosts /var/lib/jenkins/.ssh/

// chown -R jenkins:jenkins /var/lib/jenkins/.ssh
// chmod 700 /var/lib/jenkins/.ssh
// chmod 600 /var/lib/jenkins/.ssh/id_rsa
