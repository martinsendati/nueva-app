pipeline {

    environment {
        APP_NAME = "nueva-app"
        APP_TAG = "${BUILD_NUMBER}"
        REGISTRY = "martooo"
        PASS = "arquitectura123"
    }

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: slave
  name: agent-pod
spec:
  containers:
  - name: agent-container
    image: tferrari92/jenkins-inbound-agent-git-npm-docker
    command:
    - sleep
    args:
    - "99"
    env:
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: volume-0
      readOnly: false
    - mountPath: /home/jenkins/agent
      name: workspace-volume
      readOnly: false
  hostNetwork: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: Never
  volumes:
  - emptyDir:
      medium: ""
    name: workspace-volume
  - hostPath:
      path: /var/run/docker.sock
    name: volume-0
'''
            defaultContainer 'agent-container'
        }
    }
  
    stages {

        stage('Clonar repo de aplicacion') {
            steps { 
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/martinsendati/nueva-app.git'
            }
        }        
         stage('buildear imagen') {
            steps {
                sh "docker build -t $REGISTRY/$APP_NAME:$APP_TAG ."
            }
        }
        stage('docker push') {
            steps {
                sh "docker login -u $REGISTRY -p $PASS"
                sh "docker push $REGISTRY/$APP_NAME:$APP_TAG "
                
            }
        }
        stage("clonar repo de infraestructura") {
          steps { 
              git branch: 'main', changelog: false, poll: false, url: 'https://github.com/martinsendati/nueva-app.git'
          }
        }
        stage("modificar deploy") {
          steps {
            sh "sed -i s/nueva-app:.*/nueva-app:$APP_TAG/g dev/deployment.yaml"
          }
        }
        stage("pushear cambios") {
          steps {
            sh "git add dev/deployment.yaml"
            sh "git config --global user.email 'martin.barrionuevo@sendati.com'"
            sh "git config --global user.name 'martinsendati'"
            sh "git commit -m 'cambio de version'"    
            withCredentials([usernamePassword(credentialsId: "credenciales-github", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                sh('git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/martinsendati/nueva-app.git')
            }   
           

            
            
         

          
          }
        }
    } 
  }
