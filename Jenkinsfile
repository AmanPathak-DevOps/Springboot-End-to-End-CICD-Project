pipeline {
    parameters {
        booleanParam(
            defaultValue: false,
            description: 'Do not Run the Upload Artifact Stage?',
            name: 'SKIP_STAGE'
        )
    }
    agent {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed' 
                // git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/Springboot-End-to-End.git'
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://3.89.227.188:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        stage('Upload Code Artifacts') {
            agent {
                docker {
                    image 'releases-docker.jfrog.io/jfrog/jfrog-cli-v2:2.2.0'
                    reuseNode true
                }
            }
            environment {
                CI = true
                ARTIFACTORY_ACCESS_TOKEN = credentials('artifactory-access-token')
            }
            steps {
                // sh 'jfrog rt upload --url http://34.228.44.32:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} target/*.jar springboot-web-app/'
                script {
                    if (params.SKIP_STAGE) {
                        sh 'jfrog rt upload --url http://34.228.44.32:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} target/*.jar springboot-web-app/'
                    } else {
                        return
                    }
                }
            }
        }
        stage('Build & Push Docker Image') {
            environment {
                DOCKER_IMAGE = "avian19/spring-docker:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Updating Deployment File') {
            environment {
                GIT_REPO_NAME = "Springboot-end-to-end"
                GIT_USER_NAME = "AmanPathak-DevOps"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "aman07pathak@gmail.com"
                        git config user.name "AmanPathak-DevOps"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        ls
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml
                        git add deployment.yml
                        git commit -m "Update deployment Image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                    '''
                }
            }
        }
    }
}