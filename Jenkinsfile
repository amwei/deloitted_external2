node {
    def app

// setting env variable so external default port does not conflict with jenkins
    withEnv(['PORT=8081']) {

    stage('Clone repository and run test') {
        checkout scm
        echo 'install dependencies' 
        sh 'npm install'
        echo 'Run tests'
        sh 'npm test'
        echo 'Tests passed - onto the next stage'
    }  
    stage('sonarqube scan') {
        def scannerHome = tool 'sonarqube';
        withSonarQubeEnv('sonarqube') { // If you have configured more than one global server connection, you can specify its name
        sh "${scannerHome}/bin/sonar-scanner"
        }
    }
    stage('Build image') {
        app = docker.build("amwei/externalevent")
    }

    stage('Push image') {
//  push the image with two tags - the incremental build number from Jenkins, -latest tag
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("v1.${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage('deploy to cluster') {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials amiecluster-1 --zone us-central1-c --project amie-events'
                echo 'Update the image'
                sh "kubectl set image deployment/events-external events-external=amwei/externalevent:v1.${env.BUILD_NUMBER} --record"
        }
    }
}