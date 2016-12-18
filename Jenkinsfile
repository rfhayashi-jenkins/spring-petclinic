#!groovy

def base_inside = "-v ${env.JENKINS_HOME}:/home/jenkins -e HOME=/home/jenkins -e JAVA_HOME=/"
def vagrant_inside = "-v ${env.JENKINS_HOME}:/vagrant -e HOME=/vagrant"

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

    try {
        stage('Deploy') {
            docker.image('buildtools-build-tools').inside(vagrant_inside) {
                withCredentials([file(credentialsId: 'petclinic_aws_config', variable: 'AWS_CONFIG_FILE')]) {
                    withCredentials([file(credentialsId: 'petclinic_aws_pk', variable: 'AWS_PRIVATE_KEY_FILE')]) {
                        sh 'rake deploy'
                    }
                }
            }
        }
    } finally {
        docker.image('buildtools-build-tools').inside(vagrant_inside) {
            withCredentials([file(credentialsId: 'petclinic_aws_config', variable: 'AWS_CONFIG_FILE')]) {
                withCredentials([file(credentialsId: 'petclinic_aws_pk', variable: 'AWS_PRIVATE_KEY_FILE')]) {
                    sh 'rake undeploy'
                }
            }
        }
    }
}
