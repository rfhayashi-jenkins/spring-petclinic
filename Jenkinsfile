#!groovy

base_inside = "-v ${env.JENKINS_HOME}:/home/jenkins -e HOME=/home/jenkins"
vagrant_inside = "-u root:root -v ${env.JENKINS_HOME}:/vagrant -e HOME=/vagrant"
def url

def runOnBuildTools(code) {
    docker.image('buildtools-build-tools').inside(base_inside) {
        code.call()
    }
}

def runOnVagrant(code) {
    docker.image('buildtools-build-tools').inside(vagrant_inside) {
        withCredentials([file(credentialsId: 'petclinic_aws_config', variable: 'AWS_CONFIG_FILE')]) {
            withCredentials([file(credentialsId: 'petclinic_aws_pk', variable: 'AWS_PRIVATE_KEY_FILE')]) {
                code.call()
            }
        }
    }
}

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Build Tools') {
        sh 'rake build_tools'
    }

    stage('Build') {
        runOnBuildTools {
            sh 'rake build'
        }

        sh 'rake docker_build'
    }

    stage('Test') {
        parallel(
            test: {
                runOnBuildTools {
                    sh 'rake test'
                }
                junit 'target/surefire-reports/*.xml'
            },
            checkstyle: {
                runOnBuildTools {
                    sh 'rake checkstyle'
                }
                step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''])
            },
            pmd: {
                runOnBuildTools {
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

        runOnVagrant {
            try {
                sh 'rake deploy'

                sh 'rake public_ip > public_ip'
                public_ip = readFile('public_ip')
                url = "http://${public_ip}:8080"

                echo "New petclinic version available for test at: ${url}"
            } catch (e) {
                sh 'rake undeploy'
                throw e
            }
        }
    }
}

stage('Integration Test') {
    try {
        node {
            git url: 'https://github.com/rfhayashi/spring-petclinic-tests.git'

            sleep 5 // waits the application spins up

            runOnBuildTools {
                sh "(cd selenium; ./gradlew -Dbase.url=${url} -DbrowserType=htmlunit test)"
            }
        }
    } catch (e) {
        node {
            checkout scm

            runOnVagrant {
                sh 'rake undeploy'
            }
        }
        throw e
    }
}

stage('Approval') {
    try {
        input "Test new petclinic version at ${url}"
    } finally {
        node {
            checkout scm

            runOnVagrant {
                sh 'rake undeploy'
            }
        }
    }
}

