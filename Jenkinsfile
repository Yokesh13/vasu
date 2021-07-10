pipeline {
    agent any
    stages {
        stage('Git') {
            steps { git 'https://github.com/Yokesh13/vasu.git' }
        }
	stage('Build') {
	            steps { sh label: '', script: 'mvn clean'
		            sh label: '', script: 'mvn install'}
		            }

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("dockerpandian/reddy") #Name for container which created as docker file in git repository
                    app.inside {
                        sh 'echo $(curl localhost:8080)'     #host address for that container
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {  #pushing to hub with credentials given in jenkins
                    docker.withRegistry('https://registry.hub.docker.com', 'Docker') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")  #tag for container for push to hub
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script { #credentials ID is nothing but user which is added in deployment server and given in jenkins as id
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull dockerpandian/hippo1:${env.BUILD_NUMBER}\""
                        try { #here docker pull name should be same as build name given in contianer
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop Hippo\"" 
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm Hippo\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name Hippo -p 8080:8080 -d dockerpandian/hippo1:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
