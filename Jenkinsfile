pipeline {
    agent {
            kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    service_name: automate-services
    service_type: REST
spec:
  containers:
  - name: dnd
    image: docker:latest
    command: 
    - cat
    tty: true
    volumeMounts: 
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: bryandollery/terraform-packer-aws-alpine
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock  
      type: Socket
"""
    }
  }
    environment {
    CREDS = credentials('bandar_aws_creds')
    AWS_ACCESS_KEY_ID = "${CREDS_USR}"
    AWS_SECRET_ACCESS_KEY = "${CREDS_PSW}"
    OWNER = 'delta'
    PROJECT_NAME = 'webapp'
    TF_NAMESPACE= 'bandar'
  }
    stages {

         stage("init") {
        
          steps {
               container('kubectl') {
              sh 'make init'
          }
        }
      }
        stage("workspace") {
        steps {
            container('kubectl') {
            sh """
terraform workspace select sandbox-k3d
if [[ \$? -ne 0 ]]; then
  terraform workspace new sandbox-k3d
fi
make init
"""
        }
        }
        }
       stage("plan") {
        
          steps {
               container('kubectl') {
              sh 'make plan'
          }
        }
      }
      stage("apply") {
         
            steps {
                 container('kubectl') {
                sh 'make apply'
            }
          }
      }
    }
}
