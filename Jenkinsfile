node {

    def app_post
    def app_ui

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image post') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        sh "cd post"
        app_post = docker.build("davarski/post")
    }

    stage('Test image post') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app_post.inside {
            sh 'echo "Tests passed"'
        }
   

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
            app_post.push("latest")
        }

    stage('Deploy') {
        sh "helm --kubeconfig=./admin.conf init --client-only --skip-refresh"
        sh "helm --kubeconfig=./admin.conf upgrade mongodb mongodb/charts/mongodb --install   

        /* sh "helm --kubeconfig=./admin.conf install --set usePassword=false --name mongodb  stable/mongodb" */
        sh "helm --kubeconfig=./admin.conf upgrade post post/charts/post --install --set image.tag=latest"
        sh "helm --kubeconfig=./admin.conf upgrade ui ui/charts/ui --install --set image.tag=latest"
}

}
    }
