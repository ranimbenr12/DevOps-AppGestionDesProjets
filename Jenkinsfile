pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        M2_HOME = '/usr/share/maven'
        PATH = "${JAVA_HOME}/bin:${M2_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ranimbenr12/DevOps-AppGestionDesProjets.git'
            }
        }
        stage('Build') {
            steps {
                dir('backend') {
                    sh 'mvn clean compile'
                }
            }
        }
    }
}
