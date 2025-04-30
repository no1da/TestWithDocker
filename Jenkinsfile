pipeline {
    agent {
        docker {
            image 'maven:3.8.5-openjdk-17'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v $HOME/.m2:/root/.m2'
        }
    }

    environment {
        MAVEN_OPTS = "-Dmaven.repo.local=.m2_repo"
        SELENOID_PORT = '4444'
        TEST_RESULTS = 'target/allure-results'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    stages {

        stage('Checkout Source') {
            steps {
                git url: 'https://github.com/no1da/TestWithDocker.git'
            }
        }

        stage('Start Selenoid') {
            steps {
                script {
                    sh '''
                    docker-compose -f docker-compose.yml up -d selenoid selenoid-ui
                    docker ps
                    '''
                    // Пауза, чтобы Selenoid запустился
                    sleep(time: 10, unit: "SECONDS")
                }
            }
        }

        stage('Run Tests in Docker') {
            steps {
                script {
                    sh '''
                    docker build -t uiautotests .
                    docker run --rm \
                      -v $PWD/target/allure-results:/app/target/allure-results \
                      --network=bridge \
                      --name=test-runner \
                      uiautotests mvn clean test
                    '''
                }
            }
        }

        stage('Publish Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: "${TEST_RESULTS}"]]
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/allure-results/**', fingerprint: true
            }
        }

    }

    post {
        always {
            echo 'Stopping containers...'
            sh 'docker-compose down'
        }

        success {
            echo 'Pipeline succeeded!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}