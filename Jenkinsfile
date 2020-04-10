#!/usr/bin/env groovy
def label = "mcrouter-build-${UUID.randomUUID().toString()}"

podTemplate(cloud: 'kubernetes', label: label,
    containers: [
        containerTemplate(name: 'docker',
            image: 'peakcloud/dind:v0.0.2',
            command: 'dockerd --host=unix:///var/run/docker.sock',
            privileged: true,
            ttyEnabled: true,
            envVars: [
              envVar(key: 'REPO_NAME', value: 'mcrouter'),
              secretEnvVar(key: 'DOCKER_PASSWORD', secretName: 'dockerhub-token', secretKey: 'token'),
            ],
        )
    ],
    serviceAccount: 'jenkins-docker-build',
    imagePullSecrets: ['dockerhub-creds']) {
    node(label) {
        stage('Init') {
            timeout(time: 3, unit: 'MINUTES') {
                checkout scm
            }
        }
        stage('Build image') {
            container('docker') {
                sh 'docker build --no-cache -t mcrouter -f mcrouter/scripts/docker/Dockerfile mcrouter/scripts'
                sh 'docker login -u peakcloud -p ${DOCKER_PASSWORD}'
                sh 'docker tag ${REPO_NAME}:latest peakcloud/${REPO_NAME}:${Branch}'
                sh 'docker push peakcloud/${REPO_NAME}:${Branch}'
                sh 'docker tag ${REPO_NAME}:latest peakcloud/${REPO_NAME}:latest'
                sh 'docker push peakcloud/${REPO_NAME}:latest'
            }
        }
    }
}

