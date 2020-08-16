pipeline {
    
    agent none
    stages {
        
        stage("worker-build"){
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when { 
                changeset "**/worker/**"    
            }
            steps{
                echo "Compiling ..."
                dir("worker"){
                    sh "mvn compile"
                }
            }
        }
        
        stage("worker-test"){
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when { 
                changeset "**/worker/**"    
            }
            steps{
                echo "Unit testing ..."
                dir("worker")
                {
                    sh "mvn clean test"
                }                
            }
        }
        
        stage("worker-package"){
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when { 
                branch "master"  
                changeset "**/worker/**"    
            }
            steps{
                echo "Packaging ..."
                dir("worker")
                {
                    sh "mvn package -DskipTest"
                }
                archiveArtifacts artifacts: "**/target/*.jar", fingerprint: true
            }
        }
        stage("worker-docker-package"){
            agent any
            when {
                branch "master"
                changeset "**/worker/**"
            }
            steps{
                echo "Packaging into docker container..."

                script {
                    docker.withRegistry("","dockerLogin"){
                        def workerImage = docker.build("grial1/workerbuild:v${env.BUILD_ID}","./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }

        stage("result-build"){
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when { 
                changeset "**/result/**"    
            }
            steps{
                echo "Compiling ..."
                dir("result"){
                    sh "npm install"
                }
            }
        }
        
        stage("result-test"){
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when { 
                changeset "**/result/**"    
            }
            steps{
                echo "Unit testing ..."
                dir("result")
                {
                    sh "npm install && npm test"
                }                
            }
        }

        stage("result-docker-package"){
            agent any
            when {
                branch "master"
                changeset "**/result/**"
            }
            steps{
                echo "Packaging into docker container..."

                script {
                    docker.withRegistry("","dockerLogin"){
                        def workerImage = docker.build("grial1/result:v${env.BUILD_ID}","./result")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }

        stage("vote-build"){
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when { 
                changeset "**/vote/**"
            }
            steps{
                echo "Building ..."
                dir("vote"){
                    sh "pip install -r requirements.txt"
                }
            }
        }
        
        stage("vote-test"){
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when { 
                changeset "**/vote/**"  
            }
            steps{
                echo "Unit testing ..."
                dir("vote"){
                    sh "pip install -r requirements.txt"
                    sh "nosetests -v"
                }                
            }
        }
        stage("vote-docker-package"){
            agent any
            when {
                branch "master"
                changeset "**/vote/**"
            }
            steps{
                echo "Packaging into docker container..."

                script {
                    docker.withRegistry("","dockerLogin"){
                        def workerImage = docker.build("grial1/vote:v${env.BUILD_ID}","./vote")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }

        stage("deploy to dev"){
            agent any
            //when {
            //    branch "master"
            //}
            steps{
                
                echo "Deploying instavoteapp with docker compose"
                sh "docker-compose up -d"
                
                }
            }
    }
    post {
        always {
            echo "Build pipeline has finished!"
            slackSend (channel: "instavot-cd", message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        failure {
            slackSend (channel: "instavot-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
        success {
            slackSend (channel: "instavot-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}