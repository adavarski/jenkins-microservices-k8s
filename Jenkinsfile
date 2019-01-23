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
        app_post = docker.build("davarski/post", "./post")
    }

   
    stage('Push image post') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
            app_post.push("latest")
        }

    }
    
    stage('Build image ui') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        app_ui = docker.build("davarski/ui", "./ui")
    }

   
    stage('Push image ui') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
            app_ui.push("latest")
        }

    }
    
    
    stage('Deploy') {
        sh "helm --kubeconfig=./admin.conf init --client-only --skip-refresh"
        sh "helm --kubeconfig=./admin.conf upgrade mongodb mongodb/charts/mongodb --install"   

        /* sh "helm --kubeconfig=./admin.conf install --set usePassword=false --name mongodb  stable/mongodb" */
        sh "helm --kubeconfig=./admin.conf upgrade post post/charts/post --install --set image.tag=latest"
        sh "helm --kubeconfig=./admin.conf upgrade ui ui/charts/ui --install --set image.tag=latest"
}

}
