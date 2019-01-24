podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
            containerTemplate(
            name: 'helm', 
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        
        stage ('Deploy') {
            container ('helm') {
                sh "/helm init --client-only --skip-refresh"
                // sh "/helm  install --set usePassword=false --name mongodb  stable/mongodb"
                sh "/helm upgrade post post/charts/post --install --set image.tag=latest"
                sh "/helm upgrade ui ui/charts/ui --install --set image.tag=latest"
            }
        }
    }
}
