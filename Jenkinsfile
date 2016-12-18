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

        sh 'rake build_docker'
    }

    stage('Test') {
        sh './mvnm checkstyle:checkstyle'
        step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])

        sh './mvnm pmd:pmd'
        step([$class: 'PmdPublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])
    }
}
