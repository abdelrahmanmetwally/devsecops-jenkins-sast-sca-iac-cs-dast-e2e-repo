pipeline {
    agent any
    tools {
        maven 'Maven_3_8_7'
    }

    stages {
        stage('Compile and Run Sonar Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn -Dmaven.test.failure.ignore verify sonar:sonar \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.projectKey=easybuggy \
                        -Dsonar.host.url=http://192.168.56.104/:9000/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                    script {
                        app = docker.build("abdo23/DevSecOpsSecuriyTests")
                    }
                }
            }
        }

        stage('Run Container Scan with Snyk') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    script {
                        try {
                            sh "snyk container test abdo23/DevSecOpsSecuriyTests --auth=$SNYK_TOKEN"
                        } catch (err) {
                            echo err.getMessage()
                        }
                    }
                }
            }
        }

        stage('Run Snyk SCA') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh "mvn snyk:test -fn -Dsnyk.token=$SNYK_TOKEN"
                }
            }
        }

        stage('Run DAST using ZAP') {
            steps {
                sh """
                    /home/abdo/Downloads/devsecops-pipeline/ZAP_2.16.1/zap.sh \
                    -port 9393 \
                    -cmd \
                    -quickurl https://www.example.com \
                    -quickprogress \
                    -quickout /home/abdo/Downloads/devsecops-pipeline
                """
            }
        }

        stage('Run Checkov') {
            steps {
                sh "checkov -s -f main.tf"
            }
        }
    }
}
