pipeline {
    agent{
        node {
            label 'agent1'
        }
    }
    stages {
        stage('Remove basic curl tool') {
            steps {
                sh 'ls -la /usr/bin/curl'

            }
        }
        stage('Clone curl repository') {
            steps {
                // Клонируем репозиторий curl с GitHub
                git url: 'https://github.com/curl/curl.git', branch: 'master'
            }
        }
        stage("installation tolls and lib dependencies"){
            steps {
                sh 'sudo apt install -y autoconf pkg-config libssl-dev libnghttp2-dev zlib1g-dev libpsl-dev build-essential libtool'
            }
        }
        stage('configurate') {
            steps{
                // sh "sed -i 's/CURL_VERSION \"8\\.11\\.1-DEV\"/CURL_VERSION \"8.11.1-SEROV\"/' include/curl/curlver.h"
                sh 'autoreconf -fi'
                sh './configure --without-ssl'
                sh 'make'
            }
        }
        stage('Run Tests and install') {
            steps {

                sh 'make test'
                sh 'sudo make install'
            }
        }
        stage('Result stage') {
            steps{
                sh './src/curl --version'
            }
        }
    }
    post {
        failure {
            // Уведомляем об ошибке сборки
            echo 'Build failed!'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}