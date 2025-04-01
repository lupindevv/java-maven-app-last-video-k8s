pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        DOCKER_REPO = 'alexthm1/java-k8s'  // Example repo name, change to actual value
        DOCKER_REPO_SERVER = 'docker.io'  // Example Docker server, change to actual value
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                   echo 'deploying docker image...'
                }
            }
        }
        stage('deploy to eks') {
            environment {
                AWS_ACCESS_KEY = credentials('jenkins_aws_access_key')  // Example region, change to actual value
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_access_secret_key')  // Example account ID, change to actual value
            }
            steps {
                script {
                   echo 'deploying docker image to eks...'
                   sh 'kubectl create deployment nginx-deployment --image=nginx'
                }
            }
        }


    }
}
