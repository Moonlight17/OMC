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
                sh 'make'
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
                // Output the version of the installed curl
                sh './src/curl --version'
            }
        }
    }
    post {
        success {
            // Archive the built executable as an artifact upon successful build
            archiveArtifacts artifacts: 'src/.libs/, src/curl', allowEmptyArchive: true  // Adjust the path to your executable
            echo 'Build succeeded and artifact archived!'
        }
        failure {
            // If the build fails, output a failure message
            echo 'Build failed!'
        }
    }
}

