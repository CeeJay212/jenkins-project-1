pipeline {
  agent any

  tools {
    nodejs "node"
  }

  stages {

    stage ('incremental version') {
      steps {
        script {
          dir('app') {
            sh 'npm version minor --no-git-tag-version'
            def packageJson = readJSON file: 'packageJson'
            def version = packageJson.version
            env.IMAGE_NAME = "$version-$BUILD_NUMBER"
          }
        }
      }
    }

    stage ('run test') {
      steps {
        dir('app') {
          sh 'npm install'
          sh 'npm run test'
        }
      }
    }

    stage ('build and push image') {
      steps {
        withCredential([usernamePassword(withCredentialsId: 'docker-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh "docker build -t ceejay212/jenkins-project:${IMAGE_NAME} ."
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh "docker push ceejay212/jenkins-project:${IMAGE_NAME}"
        }
      }
    }

    stage ('commit version update') {
      steps {
        script {
          withCredentials([usernamePassword(withCredentialsId: 'github-cred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
            sh '''
                git config --global user.email "jenkins@example.com"
                git config --global user.name "jenkins"
                git set-url origin https://$GIT_USER:$GIT_PASS@github.com/CeeJay212/jenkins-project-1.git
                git add .
                git commit -m "version bump"
                git push https://$GIT_USER:$GIT_PASS@github.com/CeeJay212/jenkins-project-1.git HEAD:main
            sh '''
          }
        }
      }
    }
  }
}