pipeline {
    agent {
        kubernetes {
            label 'template-builder-' + UUID.randomUUID().toString().substring(0, 6)
            yamlFile 'jenkins_slave.yml'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        ansiColor('xterm')
        timestamps()
    }
    parameters {
        choice name: 'template', choices: ['jenkins/slave/base_image', 'jenkins/slave/build_template', 'jenkins/master'], description: 'Template to build'
    }
    environment {
        docker_registry = "us.gcr.io"
        project_id = "stoked-monitor-275907"
        credentials = "google_secret"
    }
    stages {
        stage('Validate template') {
            steps {
                container('build') {
                    dir(template) {
                        sh "packer validate packer.json"
                    }
                }
            }
        }
        stage('Build template') {
            steps {
                container('build') {
                    dir(template) {
                        script {
                            def hash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
                            withCredentials([string(credentialsId: credentials, variable: 'google_key')]) {
                                sh "packer build " +
                                    "-var docker_registry=${env.docker_registry} " +
                                    "-var project_id=${env.project_id} " +
                                    "-var google_key='${google_key}' " +
                                    "-var version='1.0.${hash.trim()}' " +
                                    "packer.json"
                            }
                        }
                    }
                }
            }
        }
    }
}
