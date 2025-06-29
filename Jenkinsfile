pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
    git branch: 'main', url: 'https://github.com/ParadigmParadoxicalPiercer/test-jenkin.git'
  }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-web-cicd .'
            }
        }
        stage('Run Container') {
            steps {
                sh 'docker rm -f my-web || true'
                sh 'docker run -d --name my-web -p 8890:80 my-web-cicd'
            }
        }
    }
}