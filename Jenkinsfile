pipeline {
  agent any

  stages {
    stage('Build Angular') {
      steps {
        script {
          docker.image('node:20-alpine').inside {
            sh '''
              npm install -g @angular/cli
              npm install
              ng build --configuration=production
            '''
            stash includes: 'dist/**', name: 'angular-build'
          }
        }
      }
    }

    stage('Build Nginx Image') {
      steps {
        script {
          docker.image('docker:24.0-cli').inside('-v /var/run/docker.sock:/var/run/docker.sock') {
            unstash 'angular-build'
            sh '''
              cp -r dist/* ./nginx-app/
              docker build -t my-angular-nginx ./nginx-app
            '''
          }
        }
      }
    }
  }
}
