node {
    checkout scm

    try {

        stage('Build Test Server') {
            sh 'ansible-playbook ansible/roles/ConfigHardware/tasks/main.yml -i openstack.py'
        }

        stage('Install Test Env') {
            sh 'ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook ansible/roles/ConfigSoftware/tasks/main.yml -i openstack.py'
        }



        stage('Run unit/integration tests') {
            sh 'make test'
        }

        stage('Build application artefacts') {
            sh 'make build'
        }

        stage('Create release environment and run acceptance tests') {
            sh 'make release'
        }
        stage('Tag and publish release image') {
            sh "make tag latest \$(git rev-parse --short HEAD) \$(git tag --points-at HEAD)"
            sh "make buildtag master \$(git tag --points-at HEAD)"
            withEnv(["DOCKER_USER=${DOCKER_USER}",
                "DOCKER_PASSWORD=${DOCKER_PASSWORD}",
                "DOCKER_EMAIL=${DOCKER_EMAIL}"]) {
                    sh "make login"
                }
        }

        sh "make publish"

        stage('Deploy release') {
            sh "printf \$(git rev-parse --short HEAD) > tag.tmp"
            def imageTag = readFile 'tag.tmp'
            build job: DEPLOY_JOB, parameters: [[
                $class: 'StringParameterValue',
                name: 'IMAGE_TAG',
                value: 'leletir/todobackend:' + imageTag
            ]]
        }
    }
    finally {
        stage 'Collect tests reports'
        step([$class: 'JUnitResultArchiver', testResults: '**/reports/*.xml'])

        stage 'Clean up'
        sh 'make clean'
        sh 'make logout'
    }

}