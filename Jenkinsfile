
pipeline {
    agent any

    stages {
        stage("pull-updates-to-dev"){
            steps {
                sshagent (credentials: ['e91user']) {
                    sh "ssh -o StrictHostKeyChecking=no root@3.80.85.66 'cd /home/e91user/public-repo && git checkout . && git checkout dev && git pull'"
                }
                sleep 2
            }
        }    
        stage("run-docker-container-on-dev"){
            steps {
                sshagent (credentials: ['e91user']) {
                    sh "ssh -o StrictHostKeyChecking=no root@3.80.85.66 'docker kill dev-container || true && docker rm dev-container || true && cd /home/e91user/public-repo && docker build -t dev-container-image . && docker run --name dev-container -d -p 80:80 dev-container-image'"
                }
                sleep 2
            }
        }
        
        stage("check-output-on-dev"){
            steps {
                script {
                    def response = httpRequest 'http://3.80.85.66'
                    def json = new JsonSlurper().parseText(response.content)
                    echo "Status: ${response.status}"
                    def int responseStatus = ${response.status}
                    if (responseStatus != 200) {
                        currentBuild.result = 'FAILED'
                    }
                }
            }
        }
        
        stage("merger-dev-to-prod"){
            steps('Merge approval') {
                //timeout(time: 2, unit: “HOURS”) {
                    input ("Merge dev to master? ")
                //}
                sshagent (credentials: ['e91user']) {
                    sh "ssh -o StrictHostKeyChecking=no root@35.245.42.254 'cd /home/e91user/public-repo/ && git pull origin dev  && git checkout . && git checkout master && git merge remotes/origin/dev && git push'"
                }
                sleep 2
            }
        }

        stage("run-docker-container-on-prod"){
            steps {
                sshagent (credentials: ['e91user']) {
                    sh "ssh -o StrictHostKeyChecking=no root@35.245.42.254 'docker kill prod-container || true && docker rm prod-container || true && cd /home/e91user/public-repo && docker build -t dev-container-image . && docker run --name prod-container -d -p 80:80 prod-container-image'"
                }
                sleep 2
            }
        }
        
        stage("check-output-on-prod"){
            steps {
                script {
                    def response = httpRequest 'http://35.245.42.254'
                    echo "Status: ${response.status}"
                    def int responseStatus = ${response.status}
                    if (responseStatus != 200) {
                        currentBuild.result = 'FAILED'
                    }
                }
            }
        }

    }
}
