#!/usr/bin/env groovy
properties([
    pipelineTriggers([
        [$class: 'GitHubPushTrigger']
    ]),
    disableConcurrentBuilds(),
    overrideIndexTriggers(false),
    buildDiscarder(
        logRotator(
            artifactDaysToKeepStr: '',
            artifactNumToKeepStr: '',
            daysToKeepStr: '',
            numToKeepStr: '10'
        )
    )
])

def deployableBranches = [
    'feature/jenkinsfile',
    'develop',
    'staging',
    'production'
]

def envForBranch = [
    'feature/jenkinsfile': 'dev',
    'develop': 'dev',
    'staging': 'stage',
    'production': 'prod'
]

def version = "${envForBranch[env.BRANCH_NAME] ?: ''}-${env.BUILD_NUMBER}"
env.PIPELINE_VERSION = version

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Test docker') {
        docker.image('python:3-slim').inside {
            sh """
                pwd
                ls
                python --version
            """
        }
        docker.image('python:2-slim').inside {
            sh """
                pwd
                ls
                python --version
            """
        }
    }

    stage('Test gradle') {
        docker.image('gradle:alpine').inside {
            sh """
                gradle test
            """
        }
    }

    stage('Publish test results') {
        junit 'build/test-results/**/*.xml'
    }

    stage('Clean workspace') {
        cleanWs()
    }
}

if (deployableBranches.contains(env.BRANCH_NAME)) {
    milestone 1
    stage('Confirm') {
        input 'Proceed?'
    }

    milestone 2
    node {
        lock('deploy') {
            stage("Deploy to ${envForBranch[env.BRANCH_NAME]}") {
                sh """
                    echo Deploy
                """
            }
        }
    }

}
