#!/usr/bin/groovy
@Library('fabric8-pipeline-library')
@Library('stayfriends-pipeline-library')

def utils = new io.fabric8.Utils()


def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

podTemplate(label: label, serviceAccount: 'jenkins',
        containers: [
                containerTemplate(
                        name: 'client',
                        image: 'fabric8/builder-clients',
                        command: 'cat',
                        ttyEnabled: true,
                        envVars: [
                                envVar(key: 'DOCKER_CONFIG', value: '/home/jenkins/.docker/'),
                                envVar(key: 'KUBERNETES_MASTER', value: 'kubernetes.default')
                        ]
                ),
                containerTemplate(
                        name: 'jnlp',
                        image: '10.3.0.169:80/f8/jenkins-jnlp-client',
                        command:'/usr/local/bin/start.sh',
                        args: '${computer.jnlpmac} ${computer.name}',
                        ttyEnabled: false,
                        envVars: [
                                envVar(key: 'DOCKER_HOST', value: 'unix:/var/run/docker.sock')
                        ]
                )
        ],
    volumes: [
            secretVolume(mountPath: '/home/jenkins/.docker', secretName: 'jenkins-docker-cfg'),
            secretVolume(mountPath: '/root/.ssh', secretName: 'jenkins-ssh-config'),
            hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {

    node(label) {

        //git GIT_URL
        checkout scm

        container('client') {
            def artemisVersion = "2.6.2"
            def imageVersion = "${artemisVersion}.${env.BUILD_NUMBER}"
            def imageNamespace = "stayfriends"
            def imageName = "activemq-artemis"
            def newImageName = "${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${imageNamespace}/${imageName}:${imageVersion}"

            env.setProperty('VERSION',imageVersion)

            sh "ls -hal src/assets/docker-entrypoint.sh"
            sh "chmod +x src/assets/docker-entrypoint.sh"
            sh "ls -hal src/assets/docker-entrypoint.sh"
            sh "docker build --build-arg ACTIVEMQ_ARTEMIS_VERSION=${artemisVersion} -t ${newImageName} src"
            sh "docker push ${newImageName}"
        }
    }
}
