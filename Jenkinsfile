podTemplate(yaml: '''
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
            - name: pvc-7148c7de-0314-427a-8b07-1138f5d4d24d
              mountPath: /mnt
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: pvc-7148c7de-0314-427a-8b07-1138f5d4d24d
              mountPath: /mnt
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          restartPolicy: Never
          volumes:
          - name: pvc-7148c7de-0314-427a-8b07-1138f5d4d24d
            persistentVolumeClaim:
              claimName: jenkins-pv-claim
          - name: kaniko-secret
            secret:
                secretName: dockercred
                items:
                - key: .dockerconfigjson
                  path: config.json    
''') {
node(POD_LABEL) {
    stage('debug') {
        //steps {
            echo env.GIT_BRANCH
            echo env.GIT_LOCAL_BRANCH
            echo env.BRANCH_NAME
            //echo scm.branches[0].name
        //}
    }
    stage('Build a gradle project') {        
        container('gradle') {
            stage('Build a gradle project') {
                sh '''
                /usr/bin/git clone 'https://github.com/shuniya0/week6.git'
                cd week6/
                chmod +x gradlew
                ./gradlew build
                mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
                '''
                }
            }
        }
        stage('feature') {
            //when {
              //expression {
                //return env.GIT_BRANCH == "origin/feature"
              //}
            //
            if (env.BRANCH_NAME == 'feature') {
            //stage("Declare Branch") {
                echo "I am a feature branch"
            //}
            //stage('Build Java Image') {
                container('kaniko') {
                    sh '''
                    echo 'FROM openjdk:8-jre' > Dockerfile
                    echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                    echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                    mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                    /kaniko/executor --context `pwd` --destination shashwat248/calculator-feature:0.1 
                    '''
                  }
            //}
            //stage("Clean code test") {
                container('gradle') {
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
            //}
            }
            else {
                echo "feature branch not found. Skipping."
            }
        }
        stage('main') {
            //when {
            //  expression {
            //    return env.GIT_BRANCH == "origin/main"
            //  }
            //}
            if (env.BRANCH_NAME == 'main') {
            //stage("Declare Branch") {
                echo "I am a main branch"
            //}
            //stage('Build Java Image') {
                container('kaniko') {
                        sh '''
                        echo 'FROM openjdk:8-jre' > Dockerfile
                        echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                        echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                        mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                        /kaniko/executor --context `pwd` --destination shashwat248/calculator:1.0
                        '''
                }
            //}
            //stage("Code coverage") {
                container('gradle') {
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
            //}
            //stage("Clean code test") {
                container('gradle') {
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
            //}
            }
            else {
                echo 'main branch not found. Skipping.'
            }
        }
}
}