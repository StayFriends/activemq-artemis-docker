#!/usr/bin/groovy
@Library('github.com/StayFriends/fabric8-pipeline-library@master')
@Library('github.com/StayFriends/stayfriends-pipeline-library@master')

def utils = new io.fabric8.Utils()


def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')

podTemplate(label: label, serviceAccount: 'jenkins', containers: [
    [name: 'client', image: 'fabric8/builder-clients', command: 'cat', ttyEnabled: true, envVars: [
            [key: 'DOCKER_CONFIG', value: '/home/jenkins/.docker/'],
            [key: 'KUBERNETES_MASTER', value: 'kubernetes.default']]],
    [name: 'jnlp', image: 'iocanel/jenkins-jnlp-client:latest', command:'/usr/local/bin/start.sh', args: '${computer.jnlpmac} ${computer.name}', ttyEnabled: false,
            envVars: [[key: 'DOCKER_HOST', value: 'unix:/var/run/docker.sock']]]],
    volumes: [
            [$class: 'SecretVolume', mountPath: '/home/jenkins/.docker', secretName: 'jenkins-docker-cfg'],
            [$class: 'SecretVolume', mountPath: '/root/.ssh', secretName: 'jenkins-ssh-config'],
            [$class: 'HostPathVolume', mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock']
    ]) {

	node(label) {

	    //git GIT_URL
	    checkout scm

		def imageVersion = "2.1.0.${env.BUILD_NUMBER}"
		def imageNamespace = "stayfriends"
		def imageName = "activemq-artemis"
      	def newImageName = "${env.FABRIC8_DOCKER_REGISTRY_SERVICE_HOST}:${env.FABRIC8_DOCKER_REGISTRY_SERVICE_PORT}/${imageNamespace}/${imageName}:${imageVersion}"

		env.setProperty('VERSION',imageVersion)

		sh "docker build -t ${newImageName} ."
		sh "docker push ${newImageName}"

	}
}
