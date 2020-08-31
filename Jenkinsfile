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
    CREDS = credentials('danya_aws_creds')
    AWS_ACCESS_KEY_ID = "${CREDS_USR}"
    AWS_SECRET_ACCESS_KEY = "${CREDS_PSW}"
    OWNER = 'delta'
    PROJECT_NAME = 'webapp'
  }
    stages {
        stage("init") {
        container('kubectl') {
            steps {
                sh 'make init'
            }
        }
        }
        stage("workspace") {
        steps {
            sh """
            terraform workspace list | grep sandbox-k3d
            if [[ \$? -ne 0 ]]; then
                terraform workspace new sandbox-k3d
            fi
            terraform workspace select sandbox-k3d
            make init
            """
        }
        }
       stage("plan") {
        container('kubectl') {
          steps {
              sh 'make plan'
          }
        }
      }
      stage("apply") {
          container('kubectl') {
            steps {
                sh 'make apply'
            }
          }
      }

    }
}