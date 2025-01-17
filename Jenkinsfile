def podtemplate = '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    type: docker
spec:
  containers:
  - image: docker
    name: docker
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
      type: File
    args:
    - sleep
    - "100000"
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''

podTemplate(label: 'docker', name: 'docker', namespace: 'tools', yaml: podtemplate, showRawYaml: false) {
    node("docker"){
        container("docker"){
            stage("Pull"){
                git 'https://github.com/ikambarov/Flaskex-docker.git'
            }

            withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'REGISTRY_PASSWORD', usernameVariable: 'REGISTRY_USERNAME')]) {
                stage("Build"){
                    sh 'docker build -t $REGISTRY_USERNAME/flaskex:$BRANCH_NAME  .'
                }

                stage("Test"){
                    sh '''
                        docker login -u $REGISTRY_USERNAME -p $REGISTRY_PASSWORD
                        docker push $REGISTRY_USERNAME/flaskex:$BRANCH_NAME
                    '''
                }
              image="${REGISTRY_USERNAME}"
              tag="${BRANCH_NAME}"
                        
            }
          
           withEnv(["image=$image","tag=$tag" ]) {
           stage("deploy helm"){
             build job: 'helm', parameters: [string(name: 'image', value: "$image/flaskex"), string(name: 'tag', value: "$tag")]
            }
           }
    }
  } 
}
