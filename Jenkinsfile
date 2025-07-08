pipeline {
  agent none

  stages {
    stage('Build Angular') {
      agent {
        docker {
          image 'node:20-alpine'
        }
      }
      steps {
        sh '''
          npm install -g @angular/cli
          npm install
          ng build --configuration=production
        '''
        stash includes: 'dist/**', name: 'angular-build'
      }
    }

    stage('Build Nginx Image') {
      agent {
        docker {
          image 'docker:24.0-cli'
          args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
      }
      steps {
        unstash 'angular-build'
        sh '''
          cp -r dist/* ./nginx-app/
          docker build -t my-angular-nginx ./nginx-app
        '''
      }
    }
  }
}
