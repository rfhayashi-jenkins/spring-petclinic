#!groovy
def base_inside = "-v ${env.JENKINS_HOME}:/home/jenkins -e HOME=/home/jenkins"

node {
    stage: 'Checkout'

    checkout scm

    stage: 'Build Tools'

    sh 'rake build_tools'

    stage: 'Build'

    docker.image('buildtools-build-tools').inside(base_inside) {
        sh 'rake war'
    }
}
