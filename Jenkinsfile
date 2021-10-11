
pipeline {
  agent {
    kubernetes {
      yaml '''
      spec:
        containers:
        - name: gradle
          image: gradle:6.3-jdk14     
     '''
    }
  }
stages {
  stage('debug') {
    steps {
        echo env.GIT_BRANCH
        echo env.GIT_LOCAL_BRANCH 
    }
  }
        stage('feature') {
            when { 
              expression {
                return env.GIT_BRANCH == "feature"
              }
            }
            stages {
              stage("Declare Branch") {
                   steps {
                    echo "I am a feature branch"
                   }
              }
              stage("Compile") {
                   steps {
                        sh "./gradlew compileJava"
                   }
              }
              stage("Unit test") {
                   steps {
                        sh "./gradlew test"
                   }
              }
              stage("Static code analysis") {
                   steps {
                        sh "./gradlew checkstyleMain"
                   }
              }
              stage("Package") {
                   steps {
                        sh "./gradlew build"
                   }
              }

              stage("Docker build") {
                   steps {
                        sh "docker build -t leszko/calculator:${BUILD_TIMESTAMP} ."
                   }
              }

              stage("Docker login") {
                   steps {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                                   usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                             sh "docker login --username $USERNAME --password $PASSWORD"
                        }
                   }
              }

              stage("Docker push") {
                   steps {
                        sh "docker push leszko/calculator:${BUILD_TIMESTAMP}"
                   }
              }

              stage("Update version") {
                   steps {
                        sh "sed  -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' calculator.yaml"
                   }
              }
          
              stage("Deploy to staging") {
                   steps {
                        sh "kubectl config use-context staging"
                        sh "kubectl apply -f hazelcast.yaml"
                        sh "kubectl apply -f calculator.yaml"
                   }
              }

              stage("Acceptance test") {
                   steps {
                        sleep 60
                        sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
                   }
              }

              stage("Release") {
                   steps {
                        sh "kubectl config use-context production"
                        sh "kubectl apply -f hazelcast.yaml"
                        sh "kubectl apply -f calculator.yaml"
                   }
              }
              stage("Smoke test") {
                  steps {
                      sleep 60
                      sh "chmod +x smoke-test.sh && ./smoke-test.sh"
                  }
              }
            }
        }
        stage('main') {
            when {
              expression {
                return env.GIT_BRANCH == "main"
              }
            }
            stages {
              stage("Declare Branch") {
                   steps {
                    echo "I am a main branch"
                   }
              }
              stage("Compile") {
                   steps {
                        sh "./gradlew compileJava"
                   }
              }
              stage("Unit test") {
                   steps {
                        sh "./gradlew test"
                   }
              }
              stage("Code coverage") {
                   steps {
                        sh "./gradlew jacocoTestReport"
                        sh "./gradlew jacocoTestCoverageVerification"
                   }
              }
              stage("Static code analysis") {
                   steps {
                        sh "./gradlew checkstyleMain"
                   }
              }
              stage("Package") {
                   steps {
                        sh "./gradlew build"
                   }
              }

              stage("Docker build") {
                   steps {
                        sh "docker build -t leszko/calculator:${BUILD_TIMESTAMP} ."
                   }
              }

              stage("Docker login") {
                   steps {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                                   usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                             sh "docker login --username $USERNAME --password $PASSWORD"
                        }
                   }
              }

              stage("Docker push") {
                   steps {
                        sh "docker push leszko/calculator:${BUILD_TIMESTAMP}"
                   }
              }

              stage("Update version") {
                   steps {
                        sh "sed  -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' calculator.yaml"
                   }
              }
          
              stage("Deploy to staging") {
                   steps {
                        sh "kubectl config use-context staging"
                        sh "kubectl apply -f hazelcast.yaml"
                        sh "kubectl apply -f calculator.yaml"
                   }
              }

              stage("Acceptance test") {
                   steps {
                        sleep 60
                        sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
                   }
              }

              stage("Release") {
                   steps {
                        sh "kubectl config use-context production"
                        sh "kubectl apply -f hazelcast.yaml"
                        sh "kubectl apply -f calculator.yaml"
                   }
              }
              stage("Smoke test") {
                  steps {
                      sleep 60
                      sh "chmod +x smoke-test.sh && ./smoke-test.sh"
                  }
              }
         }
        }
    }
}

