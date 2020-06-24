stage('test') {
    node ('docker-jenkins-slave'){
        // git 'https://github.com/Ytt-h/rsvpapp.git'
        checkout scm
        sh 'chmod a+x ./run_test.sh'
        sh './run_test.sh'
    }
}

node() {
//    git 'https://github.com/Ytt-h/rsvpapp.git'
    checkout scm
    stage('build the image') {
        withDockerServer([credentialsId: 'b3272bf3-8dca-4d3a-8d37-edbcf90539aa', uri: "tcp://${DOCKERHOST}:2376"]) {
            docker.build 'docyutaker/rsvpapp:mooc'
        }
    }
    stage('push the image to DockerHub') {
        withDockerServer([credentialsId: 'b3272bf3-8dca-4d3a-8d37-edbcf90539aa', uri: "tcp://${DOCKERHOST}:2376"])
        {
            withDockerRegistry([credentialsId: 'docyutaker']) {
                docker.image("${DOCKER_REGISTRY_USER}/rsvpapp:mooc").push()
            }
        }
    }
    
    stage('deploy the image to staging server'){
        withDockerServer([credentialsId: 'staging-server', uri: "tcp://${STAGING}:2376"]) {
            sh 'docker-compose -p rsvp_staging up -d'
        }
        input "Check application running at http://${STAGING}:5000 Looks good ?"
        withDockerServer([credentialsId: 'staging-server', uri: "tcp://${STAGING}:2376"]) {
            sh 'docker-compose -p rsvp_staging down -v'
        }
    }
    
    stage('deploy in production'){
        withDockerServer([credentialsId: 'production', uri: "tcp://${PRODUCTION}:2376"]) {
            sh 'docker stack deploy -c docker-stack.yaml myrsvpapp'
        }
    }
}
