
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'imalfas/maker'
        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonarscanner'
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/imalfas/maker-file.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checkout completed successfully!'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"
                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=maker-key \
                            -Dsonar.projectName=maker-project \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        def appImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {
                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    withCredentials([gitUsernamePassword(
    credentialsId: "${GIT_CREDENTIALS_ID}",
    gitToolName: 'Default'
)]) {
    sh """
        sed -i '/name: maker/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' manifests/deployment.yaml

        git -c user.name='imalfas' -c user.email='muhammadalfas9946@gmail.com' commit -am "chore(manifest): update maker image tag to ${env.BUILD_NUMBER} [skip ci]"

        git push origin ${MANIFEST_BRANCH}
    """
}
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Manifest repo updated with new image tag.'
        }
        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
