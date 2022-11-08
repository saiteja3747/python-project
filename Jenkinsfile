pipeline {
  agent {
    kubernetes {
      label 'python-dev'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: snyk-python
    image: snyk/snyk-cli:python-3
    command:
    - /bin/cat
    tty: true
  - name: snyk-docker
    image: snyk/snyk-cli:docker
    command:
    - /bin/cat
    tty: tru
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  volumes:
  - name: dind-storage
    emptyDir: {}
"""
    }
  }
   stages {
        stage('Cloning Git') {
            steps {
                 'https://github.com/saiteja3747/python-project.git'
            }
        }
  stages {
    stage('Snyk Scan') {
      failFast true
      environment {
        SNYK_TOKEN = credentials('74285fbf-bad6-479c-9c3a-707290bbcdf9')
      }	
      parallel {
        stage('dependency scan') {
          steps {
            container('snyk-python') {
              sh """
                pip install -r requirements.txt
                snyk auth ${'74285fbf-bad6-479c-9c3a-707290bbcdf9'}
                snyk test --json \
                  --file=requirements.txt \
                  --severity-threshold=high \
                  --org=cvega \
                  --project-name=project-python
              """
            }
          }
        }
        stage('docker scan') {
          steps {
            container('snyk-docker') {
              sh """
                docker build -t python-project .
                snyk auth ${SNYK_TOKEN}
                snyk test --json \
                  --docker python-project:latest \
                  --file=Dockerfile \
                  --severity-threshold=high \
                  --org=cvega \
                  --project-name=project-python
              """
            }
          }
        }
      }
    }
  }
}
