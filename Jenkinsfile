pipeline {
    
    agent none

    stages{
        stage('worker-build') {
            agent {
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/**'
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('worker-test') {
            agent {
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/**'
            }
            steps {
                echo 'Running Unit Tests on Worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker-package') {
            agent {
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                branch 'master'
                changeset '**/worker/**'
            } 
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker-docker-package') {
            agent any
            environment {
                PARSED_BRANCH_NAME = sh(script: "echo ${BRANCH_NAME} | sed -e 's#/#-#g'", returnStdout: true).trim()
            }
            when {
                changeset '**/worker/**'
            }
            steps {
                echo 'Packaging worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("moisolinux/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        sh 'echo "parsed branch name is: ${PARSED_BRANCH_NAME}"'
                        workerImage.push("${PARSED_BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }

        stage('result-build'){
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Compiling result app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('result-test'){
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Running Unit Tests on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
       stage('result-docker-package') {
            agent any
            environment {
                PARSED_BRANCH_NAME = sh(script: "echo ${BRANCH_NAME} | sed -e 's#/#-#g'", returnStdout: true).trim()
            }
            when {
                changeset '**/result/**'
            }
            steps {
                echo 'Packaging result app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("moisolinux/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        sh 'echo "parsed branch name is: ${PARSED_BRANCH_NAME}"'
                        resultImage.push("${PARSED_BRANCH_NAME}")
                        resultImage.push("latest")
                    }
                }
            }
        }

        stage('vote-build'){
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps{
                echo 'Compiling vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote-test'){
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when{
                changeset "**/vote/**"
            }
            steps{
                echo 'Running Unit Tests on vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-docker-package') {
            agent any
            environment {
                PARSED_BRANCH_NAME = sh(script: "echo ${BRANCH_NAME} | sed -e 's#/#-#g'", returnStdout: true).trim()
            }
            when {
                changeset '**/vote/**'
            }
            steps {
                echo 'Packaging vote app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("moisolinux/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        sh 'echo "parsed branch name is: ${PARSED_BRANCH_NAME}"'
                        voteImage.push("${PARSED_BRANCH_NAME}")
                        voteImage.push("latest")
                    }
                }
            }
        }

    }
 
    post {
        always {
            echo 'Building multibranch pipeline for resut run is completed...'
        }
        failure {
            slackSend (channel: 'instavote-cd', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: 'instavote-cd', message: "Build Succeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}
