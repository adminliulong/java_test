pipeline {
  agent {
    node {
      label 'maven'
    }

  }
      parameters {
        string(name: 'TAG_NAME', defaultValue: '', description: '')
    }

    environment {
        // 推送镜像的时候使用的凭证id
        DOCKER_CREDENTIAL_ID = 'unicom-harbor'
        // Kubernetes部署时使用的凭证id
        KUBECONFIG_CREDENTIAL_ID = 'uat-kubeconfig'
        // 这里使用的是我的 本地镜像仓库 的service，实际可以改成仓库所在的地址 或者集群内地址
        REGISTRY = 'harbor.jsbnb.top:30443'
        // 镜像仓库命名空间
        DOCKERHUB_NAMESPACE = 'kubesphere'
    }
  stages {
    stage('checkout scm') {
            steps {
                checkout(scm)
            }
        }
    stage('项目编译') {
      agent none
      steps {
        container('maven') {
          sh 'mvn clean package -Dmaven.test.skip=true'
          sh '''ls -ltr
ls -ltr target/
ls -ltr deploy/'''
        }

      }
    }

    stage('build & push') {
      agent none
      steps {
        container('maven') {
          sh 'docker build -t demotest:v1 -f src/main/Dockerfile .'
        }

      }
    }

    stage('push with tag') {
      agent none
      steps {
        container('maven') {
          withCredentials([usernamePassword(credentialsId : 'unicom-harbor' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh '''echo "$DOCKER_USERNAME"
echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'''
            sh 'docker tag demotest:v1 $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$BUILD_NUMBER'
            sh 'docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$BUILD_NUMBER'
          }

        }

      }
    }

    stage('部署到演示集群') {
      agent none
      steps {
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: 'uat-kubeconfig', configs: 'deploy/**')
      }
    }

  }
}
