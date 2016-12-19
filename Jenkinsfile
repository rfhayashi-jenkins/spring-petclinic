#!groovy

def base_inside = "-v ${env.JENKINS_HOME}:/home/jenkins -e HOME=/home/jenkins -e JAVA_HOME=/"
def vagrant_inside = "-u root:root -v ${env.JENKINS_HOME}:/vagrant -e HOME=/vagrant"

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
    }

    stage('Test') {
        parallel(
            test: {
                docker.image('buildtools-build-tools').inside(base_inside) {
                    sh 'rake test'
                }
                junit 'target/surefire-reports/*.xml'
            },
            checkstyle: {
                docker.image('buildtools-build-tools').inside(base_inside) {
                    sh 'rake checkstyle'
                }
                step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])
            },
            pmd: {
                docker.image('buildtools-build-tools').inside(base_inside) {
                    sh 'rake pmd'
                }
                step([$class: 'PmdPublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])
            }
        )
    }

    stage('Deploy') {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            sh 'rake docker_push'
        }

        docker.image('buildtools-build-tools').inside(vagrant_inside) {
            withCredentials([file(credentialsId: 'petclinic_aws_config', variable: 'AWS_CONFIG_FILE')]) {
                withCredentials([file(credentialsId: 'petclinic_aws_pk', variable: 'AWS_PRIVATE_KEY_FILE')]) {
                    try {
                        sh 'rake deploy'

                        sh 'rake public_ip > public_ip'
                        def output = readFile('public_ip')
                        echo output
                    } catch (e) {
                        sh 'rake undeploy'
                        throw e
                    }
                }
            }
        }
    }
}

stage('Approval') {
    try {
        input "Test new petclinic version"
    } finally {
        node {
            checkout scm

            docker.image('buildtools-build-tools').inside(vagrant_inside) {
                withCredentials([file(credentialsId: 'petclinic_aws_config', variable: 'AWS_CONFIG_FILE')]) {
                    withCredentials([file(credentialsId: 'petclinic_aws_pk', variable: 'AWS_PRIVATE_KEY_FILE')]) {
                        sh 'rake undeploy'
                    }
                }
            }
        }
    }
}

