pipeline {
    agent any

    tools {
        maven 'maven'
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
                sh 'docker build -t tomcatimage2 .'
            }
        }

        stage("image scan") {
            steps {
                sh 'trivy image tomcatimage2'
            }
        }

        stage("Docker tag") {
            steps {
                sh 'docker tag tomcatimage2 dudishivaram/java:web2'
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
                sh 'docker push dudishivaram/java:web2'
            }
        }

        stage("container") {
            steps {
                sh 'docker run -itd -p 1111:8080 dudishivaram/java:web2'
            }
        }

    }
}
