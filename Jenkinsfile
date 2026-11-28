pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        M2_HOME = '/usr/share/maven'
        PATH = "${JAVA_HOME}/bin:${M2_HOME}/bin:${env.PATH}"
        APP_DIR = 'backend/backend'
        IMAGE_NAME = 'backend-app'
        IMAGE_TAG = 'latest'
        REGISTRY = 'localhost:8083'
        NETWORK = 'devops-net'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ranimbenr12/DevOps-AppGestionDesProjets.git'
            }
        }

        stage('Build & Test Maven') {
            steps {
                dir("${APP_DIR}") {
                    sh 'mvn -B clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir("${APP_DIR}") {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials',
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    sh 'echo "$NEXUS_PASS" | docker login $REGISTRY -u "$NEXUS_USER" --password-stdin'
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh """
                    docker network create ${NETWORK} || true
                    docker rm -f mysql || true
                    docker run -d --name mysql --network ${NETWORK} \
                        -e MYSQL_ROOT_PASSWORD=root \
                        -e MYSQL_DATABASE=test_db \
                        -v mysql-data:/var/lib/mysql \
                        -p 3306:3306 mysql:8.0
                    timeout 180 sh -c 'until docker exec mysql mysqladmin ping -uroot -proot --silent; do sleep 3; done'
                """
            }
        }

        stage('Deploy backend-app') {
            steps {
                sh """
                    docker rm -f backend-app || true
                    docker pull ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker run -d --name backend-app --network ${NETWORK} \
                        -p 8085:8080 \
                        -e SPRING_DATASOURCE_URL='jdbc:mysql://mysql:3306/test_db?createDatabaseIfNotExist=true' \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e SPRING_DATASOURCE_PASSWORD=root \
                        ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Verification') {
            steps {
                sh 'docker ps'
                sh 'timeout 120 sh -c "until docker logs backend-app 2>&1 | grep -q Started; do sleep 5; done"'
                sh 'docker logs backend-app --tail 50'
            }
        }
    }

    post {
        success { echo 'Pipeline CD terminé avec succès' }
        failure { echo 'Pipeline en échec : consulter les logs de la étape en erreur' }
    }
}
EOF
