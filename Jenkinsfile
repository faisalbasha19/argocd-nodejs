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
        sh 'export GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no"'    
        git branch: 'main', credentialsId: 'gitssh-1', url: 'git@github.com:faisalbasha19/argocd-nodejs.git'
    }

    stage('Update GITHUB') {
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {   
                    //sshagent (credentials: ['gitssh-1']) {                  
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        //sh "curl -O https://gist.githubusercontent.com/m14t/3056747/raw/2e0d5df6d141400640c8f768e5c3603533812cf6/fix_github_https_repo.sh"
                        //sh "chmod +x fix_github_https_repo.sh"
                        //sh "./fix_github_https_repo.sh" 
                        //sh "cat deployment.yml"
                        sh "sed -i 's+qa-docker-nexus.mtnsat.io/dockerrepo/nodejs-app:[[:digit:]]+qa-docker-nexus.mtnsat.io/dockerrepo/nodejs-app:${DOCKERTAG}+g' deployment.yml"
                        //sh "cat deployment.yml"
                        sh "ssh-keygen -t rsa -b 4096 -C "faisal.basha@anuvu.com" -f gocd-agent-ssh -P ''"
                        sh "chmod 700 .ssh"
                        sh "ssh -T git@github.com"
                        sh "git config user.email faisal.basha@anuvu.com"
                        sh "git config user.name faisalbasha19"
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job update manifest: ${env.BUILD_ID}'"
                        sh "git branch -M main"
                        //sh "git ls-remote -h git@github.com:faisalbasha19/argocd-nodejs.git HEAD -y"                      
                        sh "git push origin main"
                        //sh "git push -f --no-verify https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/argocd-nodejs.git HEAD:main"
      }
    }
  }
}
}
}
