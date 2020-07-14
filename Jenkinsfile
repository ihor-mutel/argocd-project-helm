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
                    sh 'echo BUILD'
                }
            }
        }
        stage('connect argo') {
            container('argocd') {

                stage('Build a Maven project') {
//                        sh 'cat /root/artifacts/art'
                    withCredentials([string(credentialsId: "argocd", variable: 'ARGOCD_AUTH_TOKEN')]) {

                        sh /* CORRECT */ '''
                          argocd --insecure --auth-token $ARGOCD_AUTH_TOKEN  --server argocd-server.argocd.svc.cluster.local:80 app set nginx-helm -p image.tag=1.19.1
                          argocd --insecure --auth-token $ARGOCD_AUTH_TOKEN  --server argocd-server.argocd.svc.cluster.local:80 app sync nginx-helm
                        '''
                    }
                }
            }


        }
    }
}