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
                    app = docker.build("dockerpandian/reddy") #Name_for_container_which_created_as_docker_file_in_git_repository
                    app.inside {
                        sh 'echo $(curl localhost:8080)'     #host_address_for_that_container
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {  #pushing_to_hub_with_credentials_given_in_jenkins
                    docker.withRegistry('https://registry.hub.docker.com', 'Docker') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")  #tag_for_container_for_push_to_hub
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
                    script { #credentials_ID_is_nothing_but_user_which_is_added_in_deployment_server_and_given_in_jenkins_as_id
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull dockerpandian/hippo1:${env.BUILD_NUMBER}\""
                        try { #here_docker_pull_name_should_be_same_as_build_name_given_in_contianer
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
