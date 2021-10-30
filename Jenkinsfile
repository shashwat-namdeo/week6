pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: gradle
            image: gradle:6.3-jdk14
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: shared-storage-pv
              mountPath: /mnt
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 9999999
            volumeMounts:
            - name: shared-storage-pv
              mountPath: /mnt
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          restartPolicy: Never
          volumes:
          - name: shared-storage-pv
            persistentVolumeClaim:
              claimName: shared-storage-pvc
          - name: kaniko-secret
            secret:
                secretName: dockercred
                items:
                - key: .dockerconfigjson
                  path: config.json    
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
  stage('Build a gradle project') {
    steps {
      sh '''
      chmod +x gradlew
      ./gradlew build
      chmod 777 /mnt
      mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
      '''
    }
  }
        stage('feature') {
            when { 
              expression {
                return env.GIT_BRANCH == "origin/feature"
              }
            }
            stages {
              stage("Declare Branch") {
                   steps {
                    echo "I am a feature branch"
                   }
              }
              stage('Build Java Image') {
                steps {
                  container('kaniko') {
                    //stage('Build a Go project') {
                      steps {
                        sh '''
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination shashwat248/calculator-feature:0.1 
                        '''
                      }
                    //}
                  }
                }
              }
              stage("Clean code test") {
                steps {
                sh '''
                pwd
                ./gradlew checkstyleMain
                '''
                publishHTML (target: [
                  reportDir: 'build/reports/checkstyle',
                  reportFiles: 'main.html',
                  reportName: "Checkstyle Report"
                ])
                }
              }
           }
      }
        stage('main') {
            when {
              expression {
                return env.GIT_BRANCH == "origin/main"
              }
            }
            stages {
              stage("Declare Branch") {
                   steps {
                    echo "I am a main branch"
                   }
              }
              stage('Build Java Image') {
                steps {
                  container('kaniko') {
                    //stage('Build a Go project') {
                      steps {
                        sh '''
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination shashwat248/calculator:1.0
                        '''
                      }
                    //}
                  }
                }
              }
              stage("Code coverage") {
                  steps {
                  sh '''
                  pwd
                  ./gradlew jacocoTestCoverageVerification
                  ./gradlew jacocoTestReport
                  '''
                  publishHTML (target: [
                    reportDir: 'build/reports/jacoco/test/html',
                    reportFiles: 'index.html',
                    reportName: "JaCoCo Report"
                  ])
                }
              }
              stage("Clean code test") {
                  steps {
                  sh '''
                  pwd
                  ./gradlew checkstyleMain
                  '''
                  publishHTML (target: [
                    reportDir: 'build/reports/checkstyle',
                    reportFiles: 'main.html',
                    reportName: "Checkstyle Report"
                  ])
                }
              }
            }
        }
  }
}
