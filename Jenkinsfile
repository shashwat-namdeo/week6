
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
                return env.GIT_BRANCH == "origin/feature"
              }
            }
            steps { 
               echo "I am a feature branch"
            }
        }
        stage('main') {
            when {
              expression {
                return env.GIT_BRANCH == "origin/main"
              }
            } 
            steps { 
               echo "I am a main branch"
            }
        }
    }
}

