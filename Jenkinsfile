podTemplate(containers: [
        containerTemplate(name: 'maven', image: 'maven:3-openjdk-11', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'alpine', image: 'alpine:latest', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'argocd', image: 'payfit/argocd-cli', ttyEnabled: true, command: 'cat')
],
        volumes: [hostPathVolume(mountPath: '/root/.m2', hostPath: '/tmp/cache'), hostPathVolume(mountPath: '/root/artifacts', hostPath: '/tmp/artifacts')]
) {

    node(POD_LABEL) {
        stage('Get a Maven project') {
            //git 'https://github.com/jenkinsci/kubernetes-plugin.git'
            container('maven') {
                stage('Build a Maven project') {
                    //sh 'mvn -B clean install'
                    sh 'env'
                    sh 'env > /root/artifacts/env'
                    sh 'echo TEST > /root/artifacts/art'
                }
            }
        }
        stage('connect argo') {
            container('argocd') {

                stage('Build a Maven project') {
//                        sh 'cat /root/artifacts/art'
                    withCredentials([string(credentialsId: "argocd", variable: 'ARGOCD_AUTH_TOKEN')]) {

                        sh /* CORRECT */ '''
                          argocd --insecure --server argocd-server.argocd.svc.cluster.local:80 app list --auth-token $ARGOCD_AUTH_TOKEN app set nginx-helm -p image.pullPolicy=Always
                          argocd --insecure --server argocd-server.argocd.svc.cluster.local:80 app list --auth-token $ARGOCD_AUTH_TOKEN app sync nginx-helm
                        '''
                    }
                }
            }


        }
    }
}