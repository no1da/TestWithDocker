pipeline {
    agent {
        docker {
            image 'maven:3.8.5-openjdk-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }

    environment {
        DOCKER_COMPOSE_VERSION = '1.29.2'
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/no1da/TestWithDocker'
            }
        }

        stage('Start Selenoid') {
            steps {
                script {
                    sh 'docker-compose -f docker-compose.yml up -d selenoid selenoid-ui'
                    sleep(time: 5, unit: "SECONDS") 
                }
            }
        }

        stage('Test Execution') {
            steps {
                script {
                    sh 'docker-compose run --rm uiautotests'
                    sh 'docker cp $(docker ps -aqf "name=uiautotests"):/app/target/allure-results ./target/allure-results || true'
                }
            }
        }

        stage('Teardown') {
            steps {
                script {
                    sh 'docker-compose down'
                }
            }
        }

        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/allure-results/**/*.*', allowEmptyArchive: true
            cleanWs()
        }
    }
}