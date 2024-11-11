pipeline {
    agent{
        node {
            label 'agent1'  // Specify that this pipeline will run on the agent
        }
    }
    stages {
        stage('Сheck basic curl tool') {
            steps {
                // Check if curl is installed on the agent
                sh 'ls -la /usr/bin/curl'
            }
        }
        stage('Clone curl repository') {
            agent { label '' } // This stage will run on the master node (as it requires Ethernet access)
            steps {
                // Clone the curl repository from GitHub
                git url: 'https://github.com/curl/curl.git', branch: 'master'
                stash name: 'curl-repo', includes: '**'

            }
        }
        // stage("installation tolls and lib dependencies"){
        //     agent { label '' } // This stage will run on the master node (as it requires Ethernet access)
        //     steps {
        //         // Install dependencies and build tools
        //         sh 'sudo apt install -y autoconf pkg-config libssl-dev libnghttp2-dev zlib1g-dev libpsl-dev build-essential libtool'
        //     }
        // }
        stage('configurate') {
            steps{
                unstash 'curl-repo'
                sh 'ls -la'
                // Prepare configuration files for building
                sh 'autoreconf -fi'
                sh './configure --disable-shared --enable-static --without-ssl'
                sh 'make clean && make'
            }
        }
        stage('Run Tests and install') {
            steps {
                // Run tests and install the built tool
                // sh 'make test'
                sh 'sudo make install'
            }
        }
        stage('Result stage') {
            steps{
                // Output the version of the installed curl and test isolate binary file
                sh './src/curl --version'
                sh 'mkdir -p ./isolate_folder_for_curl_binary_file'
                sh 'cp ./src/curl ./isolate_folder_for_curl_binary_file/curl'
                sh './isolate_folder_for_curl_binary_file/curl --version'
            }
        }
    }
    post {
        success {
            // Archive the built executable as an artifact upon successful build
            archiveArtifacts artifacts: 'src/curl', allowEmptyArchive: true  // Adjust the path to your executable
            script {
                mess = "✅✅✅ #${env.JOB_NAME}  \n\n *Successfully* builded \n\n It was build number (${env.BUILD_NUMBER})"
            }
            withCredentials([string(credentialsId: 'dest', variable: 'SECRET')]){
                echo 'Build succeeded and artifact archived!'
                telegramSend(message: "${mess}", chatId: "${dest}")
            }
        }
        failure {
            // If the build fails, output a failure message
            script {
                mess = "🚨🚨🚨 #${env.JOB_NAME} Build with \n\n *ERROR* \n\n ${env.BUILD_NUMBER}, @Moonlight1790"
            }
            withCredentials([string(credentialsId: 'dest', variable: 'SECRET')]){
                echo 'Build failed!'
                telegramSend(message: "${mess}", chatId: "${dest}")
            }
        }
    }
}
