
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
      stage('Build a gradle project') {
        setps {
          sh '''
          cd Chapter08/sample1
          chmod +x gradlew
          ./gradlew test
          '''
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
              stage("Clean code test") {
                steps {
                sh '''
                pwd
                cd Chapter08/sample1
                ./gradlew checkstyleMain
                '''
                publishHTML (target: [
                  reportDir: 'Chapter08/sample1/build/reports/checkstyle',
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
                return env.GIT_BRANCH == "main"
              }
            }
            stages {
              stage("Declare Branch") {
                   steps {
                    echo "I am a main branch"
                   }
              }
              stage("Code coverage") {
                  setps {
                  sh '''
                  pwd
                  cd Chapter08/sample1
                  ./gradlew jacocoTestCoverageVerification
                  ./gradlew jacocoTestReport
                  '''
                  publishHTML (target: [
                    reportDir: 'Chapter08/sample1/build/reports/jacoco/test/html',
                    reportFiles: 'index.html',
                    reportName: "JaCoCo Report"
                  ])
                  }
              }
              stage("Clean code test") {
                  setps {
                  sh '''
                  pwd
                  cd Chapter08/sample1
                  ./gradlew checkstyleMain
                  '''
                  publishHTML (target: [
                    reportDir: 'Chapter08/sample1/build/reports/checkstyle',
                    reportFiles: 'main.html',
                    reportName: "Checkstyle Report"
                  ])
                }
              }
            }
            }
        }
}

