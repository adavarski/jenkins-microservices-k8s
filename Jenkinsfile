node {

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Deploy') {
        sh "helm --kubeconfig=./admin.conf init --client-only --skip-refresh"
        sh "helm --kubeconfig=./admin.conf install --set usePassword=false --name mongodb  stable/mongodb"
        sh "helm --kubeconfig=./admin.conf upgrade post post/charts/post --install --set image.tag=latest"
        sh "helm --kubeconfig=./admin.conf upgrade ui ui/charts/ui --install --set image.tag=latest"
}

}
