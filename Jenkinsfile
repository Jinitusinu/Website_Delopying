pipeline {
    agent any
    stages {
        stage('Pull Image') {
            steps {
                sshagent(['docker-server']) {
                    sh 'ssh root@192.168.252.20 "docker pull jinitus/static-website-nginx:latest"'
                }
            }
        }

        stage('Stop and Remove Existing Container') {
            steps {
                sshagent(['docker-server']) {
                    script {
                        // Ensuring no container is using port 8082
                        echo "Stopping and removing any existing containers using port 8082"
                        sh '''
                            ssh root@192.168.252.20 "docker stop main-container || true"
                            ssh root@192.168.252.20 "docker rm main-container || true"
                            # If any container is using port 8082, stop and remove it
                            ssh root@192.168.252.20 "docker ps -q -f 'ancestor=jinitus/static-website-nginx:latest' -f 'publish=8082' | xargs -r docker stop"
                            ssh root@192.168.252.20 "docker ps -q -f 'ancestor=jinitus/static-website-nginx:latest' -f 'publish=8082' | xargs -r docker rm"
                        '''
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                sshagent(['docker-server']) {
                    sh '''
                        # Run the container with the new image
                        ssh root@192.168.252.20 "docker run --name main-container -d -p 8082:80 jinitus/static-website-nginx:latest"
                    '''
                }
            }
        }

        stage('Test Website') {
            steps {
                sh 'curl -I http://192.168.252.20:8082 || exit 1'
            }
        }
    }
}
