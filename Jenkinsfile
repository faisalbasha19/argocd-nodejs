podTemplate(yaml: '''
              apiVersion: v1
              kind: Pod
              spec:
                volumes:
                - name: docker-socket
                  emptyDir: {}
                containers:
                - name: docker
                  image: docker:19.03.1
                  readinessProbe:
                    exec:
                      command: [sh, -c, "ls -S /var/run/docker.sock"]
                  command:
                  - sleep
                  args:
                  - 99d
                  volumeMounts:
                  - name: docker-socket
                    mountPath: /var/run
                - name: docker-daemon
                  image: docker:19.03.1-dind
                  securityContext:
                    privileged: true
                  volumeMounts:
                  - name: docker-socket
                    mountPath: /var/run
                - name: node
                  image: node:latest
                  command:
                  - cat
                  tty: true                  
''') {  
    
node(POD_LABEL) {
    def app

    stage('Clone repository') {
      

        checkout scm
    }

    stage('Update GITHUB') {
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {   
                    withCredentials([usernamePassword(credentialsId: 'gitssh-1', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh "git config user.email faisal.basha@anuvu.com"
                        sh "git config user.name faisalbasha19"
                        sh "cat deployment.yml"
                        sh "sed -i 's+qa-docker-nexus.mtnsat.io/dockerrepo/nodejs-app:*+qa-docker-nexus.mtnsat.io/dockerrepo/nodejs-app:${DOCKERTAG}+g' deployment.yml"
                        sh "cat deployment.yml"
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job update manifest: ${env.BUILD_ID}'"
                        sh "git push --force https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/argocd-sample-nodejs.git HEAD:main"                        
      }
    }
  }
}
}
}
