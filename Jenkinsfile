node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("polaaditya/hellonode")
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
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    stage('Deploy New Stack') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
         withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-authentication']]) {
           sh 'aws cloudformation create-stack --region us-east-1 --stack-name myapp-stack-${BUILD_NUMBER} --template-body file://aws-cft.yaml'
           sleep 30
           sh 'aws cloudformation describe-stacks --region us-east-1 --stack-name myapp-stack-${BUILD_NUMBER}'
           def APP_URL = sh 'aws cloudformation describe-stacks --region us-east-1 --stack-name myapp-stack-${BUILD_NUMBER} | grep OutputValue | cut -d\':\' -f2 | tr -d \'",\''
           echo $APP_URL
           echo '######################################################'
           sh 'echo "Your Node JSApplication is running on ${APP_URL}:8000"'
           echo '######################################################'
          }

    }
    stage('Delete Old Stack') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
         withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-authentication']]) {
           PREV_BUILD = sh 'expr ${BUILD_NUMBER} - 1'
           sh 'aws cloudformation delete-stack --region us-east-1 --stack-name myapp-stack-${PREV_BUILD}'
          }

    }
}
