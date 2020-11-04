pipeline {

    agent any

    tools {
        gradle 'gradle'
    }

    stages {

        stage('gradle build') {
            steps {
                sh 'chmod +x gradlew && ./gradlew build'
            }
        
        post {
            success {
                slackSend(color: 'good', message: "Gradle project '${JOB_NAME}' [${GIT_BRANCH}] has been updated and pulled from Github.")
                }
            failure {
                slackSend(color: 'danger', message: "Gradle project '${JOB_NAME}' [${GIT_BRANCH}] failed in building.")
                }
            }
        }


        stage('sonarqube quality check') {
            steps {
                withSonarQubeEnv("SonarCloud")
                {
                sh "./gradlew sonarqube -Dsonar.branch.name=\"master\""
                sleep(10)
                }
            }
        }

        stage('sonar quality gate') {
            steps {
                sh 'echo well'
                // timeout(time: 1, unit: 'HOURS') {
                //     waitForQualityGate abortPipeline: true
                // }
            }
            post {
                success {
                    slackSend(color: 'good', message: "Gradle project '${JOB_NAME}' [${GIT_BRANCH}] has passed the SonarQube quality gate.")
                }
                failure {
                    slackSend(color: 'danger', message: "Gradle project '${JOB_NAME}' [${GIT_BRANCH}] failed the Sonar quality gate.")
                }
            }
        }

        stage('docker build and tag') {
            steps {
                sh "docker build -t dougliu/challenge2web:${currentBuild.number} ."
                sh "docker tag dougliu/challenge2web:${currentBuild.number} dougliu/challenge2web:latest"
            }
        }

        stage('docker push to dockerhub') {
            steps {
                withDockerRegistry([credentialsId: 'DockerCred', url: '']) {
                    sh "docker push dougliu/challenge2web:${currentBuild.number}"
                    sh "docker push dougliu/challenge2web:latest"
                }
            }
            post {
                success {
                    slackSend(color: 'good', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] new image is built and pushed to Dockerhub.")
                    }
                failure {
                    slackSend(color: 'danger', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] failed push the new image to dockerhub.")
                    }
                }
            }

        stage('deploy to remote K8S cluster') {
            steps {
                withKubeConfig([credentialsId: 'k8sCred', serverUrl: '${KUBEURL}']) {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
                 }
             }
            post {
                success {
                    slackSend(color: 'good', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] deployment to K8S is successful")
                }
                failure {
                    slackSend(color: 'danger', message: "The project '${JOB_NAME}' [${GIT_BRANCH}] failed to deployer to k8s")
                }
            }
        }
    }
}