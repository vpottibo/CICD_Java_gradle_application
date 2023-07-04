pipeline {
    agent any
    environment {
        VERSION="${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps {
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage ("docker build & docker push") {
            steps {
                script {
                   withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                            docker build -t 34.28.133.24:8083/springapp:${VERSION} .
                            docker login -u admin -p ${docker_password} 34.28.133.24:8083
                            docker push 34.28.133.24:8083/springapp:${VERSION}
                            docker rmi 34.28.133.24:8083/springapp:${VERSION}
                        '''
                   }
                }
            }
        }
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=7db607d9-ac51-47fc-8edb-643b85ac23eb']) {
                              sh 'helm datree test myapp/'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo "SUCCESS"
        }
    }
}