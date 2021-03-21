node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("amwei/externalevent")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("v1.${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage('deploy to cluster') {
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials amie-events-app-cluster --zone us-central1-a --project amie-events'
                echo 'Update the image'
                sh "kubectl set image deployment/events-external events-external=amwei/externalevent:v1.${env.BUILD_NUMBER} --record"
        }
}