pipeline {
    agent any

    environment {
        REGISTRY = 'solofonore/html'
        DOCKER_IMAGE = "${REGISTRY}:version-${env.BUILD_ID}"
        VERSION_FILE = 'version.txt'
        DEFAULT_VERSION = '1'
        VERSION_NUMBER = ''
    }

    stages {
        stage('Clonage du dépôt') {
            steps {
                sh "git clone https://github.com/solofo772/projet.git && cd projet/"
                sh 'ls $pwd'
            }
        }

        stage('Récupération de la version') {
            steps {
                sh '''
                    if [ -f "${VERSION_FILE}" ]; then
                        VERSION_NUMBER=$(cat "${VERSION_FILE}" | tr -d '[:space:]')
                    else
                        VERSION_NUMBER="${DEFAULT_VERSION}"
                    fi
                '''
            }
        }

        stage('Construction de l\'image Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Déploiement dans Kubernetes') {
            steps {
                script {
                    def manifestPath = "${pwd()}/manifest"
                    sh "kubectl apply -f ${manifestPath}/deployment.yaml"
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_IMAGE}"
                    sh "docker run -d -t -p 80:80 --name web ${DOCKER_IMAGE}"

                }
            }
        }
    }
}
