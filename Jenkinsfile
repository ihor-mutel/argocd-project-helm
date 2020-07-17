podTemplate(containers: [
        containerTemplate(name: 'maven', image: 'maven:3-openjdk-11', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'argocd-cli', image: 'payfit/argocd-cli', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kaniko', image: 'gcr.io/kaniko-project/executor:debug-539ddefcae3fd6b411a95982a830d987f4214251', ttyEnabled: true, command: 'cat')
],
        volumes: [
                hostPathVolume(mountPath: '/root/.m2', hostPath: '/tmp/cache'),
                hostPathVolume(mountPath: '/root/artifacts', hostPath: '/tmp/artifacts'),
                secretVolume(secretName: 'migor', mountPath: '/kaniko/.docker')
        ]
) {

    node(POD_LABEL) {
        stage('Build docker') {
            stage('Get a Maven project') {
                git 'https://github.com/jenkinsci/kubernetes-plugin.git'
                container('maven') {
                    stage('Build a Maven project') {
                        sh 'mvn -B clean install'

                    }
                }
            }
            container('kaniko') {
                git 'https://github.com/web1991t/argocd-project-helm.git'
                stage('Build docker') {

                    sh /* BUILD AND PUSH IMAGE */ '/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination=migor/test:latest'
                }
            }
        }

        stage('Deploy using argocd') {
            container('argocd-cli') {

                stage('update version and deploy') {
                    withCredentials([string(credentialsId: "argocd", variable: 'ARGOCD_AUTH_TOKEN')]) {
                        //sh 'sleep 100000000'
                        sh /* UPDATE VERSION */ '''
                          argocd --insecure --auth-token $ARGOCD_AUTH_TOKEN  --server argocd-server.argocd.svc.cluster.local:80 app set nginx-helm -p image.tag=1.19.1
                        '''

                        sh /* SYNC */ '''
                          argocd --insecure --auth-token $ARGOCD_AUTH_TOKEN  --server argocd-server.argocd.svc.cluster.local:80 app sync nginx-helm
                        '''

                    }
                }
            }


        }
    }
}