pipeline {
    // Chaque stage choisit son propre env
    agent none
    stages {
        stage('Build Maven') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17-alpine'
                    args '-u root -v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh "mvn clean package -DskipTests"
            }
        }
        stage('Unit Test') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17-alpine'
                    args '-u root -v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh "mvn test"
            }
        }
        stage('Push to Docker Hub') {
            agent {
                docker {
                    image 'docker:25.0.3'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh "docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_PASSWORD"
                    sh "docker build -t moustapham/mini-api:v$BUILD_NUMBER ."
                    sh "docker push moustapham/mini-api:v$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy on Remote Server') {
            agent any
            steps {
                script {
                    // Demande de confirmation avant d'exécuter le déploiement
                    def userInput = input(
                        message: 'Voulez-vous déployer sur le serveur distant ?',
                        ok: 'Déployer'
                    )

                    if (userInput != null) {
                        // sshagent(['remote_ssh_key']) { //ou bien installer le plugin sshagent
                        // /idrsa doit etre montee comme un volume (cle privee Jenkins -> serveur distant)
                        // TODO: remplace user@remote.server.com par l'adresse reelle du serveur cible
                        sh """
                            ssh -i /idrsa -o StrictHostKeyChecking=no user@remote.server.com '
                            cd /home/user/app &&
                            docker pull moustapham/mini-api:v$BUILD_NUMBER &&
                            docker stop mini-api || true &&
                            docker rm mini-api || true &&
                            docker run -d --name mini-api -p 8080:8080 moustapham/mini-api:v$BUILD_NUMBER
                            '
                        """
                        // }
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline build successfuly"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
