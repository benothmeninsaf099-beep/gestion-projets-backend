pipeline {
    agent any

    environment {
        REGISTRY = 'localhost:5000'
        IMAGE    = 'backend-app'
        TAG      = 'latest'
        NETWORK  = 'devops-net'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/benothmeninsaf099-beep/gestion-projets-backend.git'
            }
        }

        stage('Build & Test Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE}:${TAG} .'
                sh 'docker tag ${IMAGE}:${TAG} ${REGISTRY}/${IMAGE}:${TAG}'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'registry-credentials',
                        usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
                    sh 'echo "$REG_PASS" | docker login ${REGISTRY} -u "$REG_USER" --password-stdin'
                    sh 'docker push ${REGISTRY}/${IMAGE}:${TAG}'
                }
            }
        }

        stage('Déploiement MySQL') {
            steps {
                sh 'docker network create ${NETWORK} || true'
                sh '''
                    if [ "$(docker ps -aq -f name=^mysql$)" ]; then
                        echo "Le conteneur mysql existe déjà : démarrage"
                        docker start mysql
                    else
                        docker run -d --name mysql --network ${NETWORK} \
                          -e MYSQL_ROOT_PASSWORD=root \
                          -e MYSQL_DATABASE=test_db \
                          -v mysql-data:/var/lib/mysql \
                          mysql:8.0
                    fi
                '''
                sh '''
                    echo "Attente du démarrage de MySQL..."
                    for i in $(seq 1 30); do
                        docker exec mysql mysqladmin ping -h localhost -uroot -proot --silent && break
                        sleep 5
                    done
                '''
            }
        }

        stage('Déploiement backend-app') {
            steps {
                sh 'docker stop backend-app || true'
                sh 'docker rm backend-app || true'
                sh 'docker pull ${REGISTRY}/${IMAGE}:${TAG}'
                sh '''
                    docker run -d --name backend-app --network ${NETWORK} -p 8089:8080 \
                      -e SPRING_DATASOURCE_URL="jdbc:mysql://mysql:3306/test_db?createDatabaseIfNotExist=true" \
                      -e SPRING_DATASOURCE_USERNAME=root \
                      -e SPRING_DATASOURCE_PASSWORD=root \
                      --restart on-failure \
                      ${REGISTRY}/${IMAGE}:${TAG}
                '''
            }
        }

        stage('Vérification') {
            steps {
                sh '''
                    echo "Attente du démarrage du backend..."
                    for i in $(seq 1 24); do
                        docker logs backend-app 2>&1 | grep -q "Started BackendApplication" && break
                        sleep 5
                    done
                '''
                sh 'docker ps'
                sh 'docker logs backend-app'
            }
        }
    }
}
