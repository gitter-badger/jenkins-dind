#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@v2.3.3') _

// Initialize global config
cfg = jplConfig('jenkins-dind', 'docker', '', [hipchat: '', slack: '', email:'redpandaci+jenkinsdind@gmail.com'])
String jenkinsVersion

pipeline {
    agent none

    stages {
        stage ('Initialize') {
            agent { label 'docker' }
            steps  {
                jplStart(cfg)
                script {
                    jenkinsVersion = readFile "${env.WORKSPACE}/src/jenkins-version"
                }
            }
        }
        stage ('Build') {
            agent { label 'docker' }
            steps {
                docker.build('redpandaci/jenkins-dind:test', '--no-cache')
                docker.build('redpandaci/jenkins-dind:latest')
            }
        }
        stage ('Test') {
            agent { label 'docker' }
            steps  {
                sh 'bin/test.sh'
            }
        }
        stage ('Release confirm') {
            when { branch 'release/v*' }
            steps {
                jplPromoteBuild(cfg)
            }
        }
        stage ('Release finish') {
            agent { label 'docker' }
            when { expression { cfg.BRANCH_NAME.startsWith('release/v') && cfg.promoteBuild.enabled } }
            steps {
                sh 'make && git add README.md && git commit -m "Docs: Update README.md with Red Panda JPL"'
                sh "docker rmi redpandaci/jenkins-dind:test redpanda-ci/jenkins-dind:${jenkinsVersion} redpandaci/jenkins-dind:latest || true"
                jplDockerPush (cfg, "redpandaci/jenkins-dind", jenkinsVersion, "", "https://registry.hub.docker.com", "redpandaci-docker-credentials")
                jplDockerPush (cfg, "redpandaci/jenkins-dind", "latest", "", "https://registry.hub.docker.com", "redpandaci-docker-credentials")
                jplCloseRelease(cfg)
            }
        }
        stage ('PR Clean') {
            agent { label 'docker' }
            when { branch 'PR-*' }
            steps {
                deleteDir()
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        skipDefaultCheckout()
        timeout(time: 1, unit: 'DAYS')
    }
}
