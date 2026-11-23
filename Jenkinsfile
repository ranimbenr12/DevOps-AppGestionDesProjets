pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'master',
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
