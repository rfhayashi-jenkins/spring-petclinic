#!groovy

def base_inside = "-v ${env.JENKINS_HOME}:/home/jenkins -e HOME=/home/jenkins -e JAVA_HOME=/"

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Build Tools') {
        sh 'rake build_tools'
    }

    stage('Build') {
        docker.image('buildtools-build-tools').inside(base_inside) {
            sh 'rake build'
        }

        sh 'rake docker_build'

        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            sh 'rake docker_push'
        }
    }

    stage('Test') {
        sh 'rake checkstyle'
        step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])

        sh 'rake pmd'
        step([$class: 'PmdPublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])
    }
}
