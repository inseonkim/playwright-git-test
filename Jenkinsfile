pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mcr.microsoft.com/playwright:v1.53.2-jammy'
        GIT_CREDENTIALS_ID = '95982574-c7ba-4eda-9f83-80244d0163e4'
        GIT_REPO_URL = 'https://github.com/inseonkim/playwright-git-test.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
                echo "env : ${params.env}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    url: "${env.GIT_REPO_URL}",
                    credentialsId: "${env.GIT_CREDENTIALS_ID}"
            }
        }

        stage('Run tests in Docker') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).inside('-u root') {
                        echo "${params.workers}"
                        sh 'npm ci'
                        sh 'npx playwright install'
                        sh "npx playwright test --workers=${params.workers}"
                    }
                }
            }
        }
    }

    post {
        always {
            // HTML 리포트 파일들을 아티팩트로 저장
            archiveArtifacts artifacts: '**/playwright-report/**, **/monocart-report/**', 
                           fingerprint: true, 
                           allowEmptyArchive: true

            // HTML Publisher로 리포트 게시
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: ''
            ])
            
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'monocart-report',
                reportFiles: 'index.html',
                reportName: 'Monocart Test Report',
                reportTitles: ''
            ])
            
            echo 'Cleaning up...'
            // HTML 리포트 게시 후에 cleanup
            cleanWs()
        }
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}