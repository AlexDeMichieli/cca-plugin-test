pipeline {
    agent any

    environment {
        NODE_VERSION = '20'
        DOCKER_REGISTRY = 'registry.example.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Publish') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry',
                    usernameVariable: 'REG_USER',
                    passwordVariable: 'REG_PASS'
                )]) {
                    sh 'echo "$REG_PASS" | docker login $DOCKER_REGISTRY -u "$REG_USER" --password-stdin'
                    sh 'docker push $DOCKER_REGISTRY/myapp:$BUILD_NUMBER'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
