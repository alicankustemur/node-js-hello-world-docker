env.CONTAINER_REGISTRY = "harbor.k8s.kandy.io"
env.GITHUB_CREDENTIALS_ID = "github-token"
env.DOCKER_CREDENTIALS_ID = "docker-credential"
env.GIT_REPO = "https://github.com/alicankustemur/node-js-hello-world-docker.git"
env.BRANCH = "master"
env.APP = "$CONTAINER_REGISTRY/kandy/nodejs"
env.WORK_DIR = "nodejs"

withCredentials([string(credentialsId: "$GITHUB_CREDENTIALS_ID",variable: 'GITHUB_TOKEN')]) {
    env.GITHUB_TOKEN = "${GITHUB_TOKEN}"
} 

node {
    stage('Git Access && And Checkout Repository') {
        checkout([$class: 'GitSCM', 
        branches: [[name: "$BRANCH"]],
        doGenerateSubmoduleConfigurations: false,
        extensions: [[$class: 'RelativeTargetDirectory',
        relativeTargetDir: "$WORK_DIR"]],
        submoduleCfg: [], userRemoteConfigs:[[url: "$GIT_REPO"]]])
    }

    stage('Build image') {
        dir("./$WORK_DIR") {
            sh 'docker build -t $APP .'
        }
    }

    stage('Push image') {
        withCredentials([
            usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID",
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASSWORD')
        ]){
            sh 'docker login $CONTAINER_REGISTRY -u $DOCKER_USER -p $DOCKER_PASSWORD' 
        }
        dir("./$WORK_DIR") {
            def COMMIT_ID = sh(script: "git log -1 --pretty=oneline | awk -F, '{printf \"%s\", substr(\$0,0,6)}'", returnStdout: true)
            def COMMIT_AUTHOR = sh(script: "git log -1 --pretty=format:'%ae' | cut -d'@' -f 1 | sed -r 's/[.]/-/g'", returnStdout: true)
            env.TAG = "$BRANCH"+"-"+ COMMIT_ID +"-"+ COMMIT_AUTHOR ;
        }
        sh 'docker tag $APP $APP:$TAG'
        sh 'docker push $APP:$TAG'
    }
    
    stage('Cleaning after from build') {
        dir("./$WORK_DIR") {
            // remove image from Jenkins disk.
            sh 'docker rmi -f $(docker images $APP:$TAG --format "{{.ID}}")'
            sh 'docker rmi $(docker images -f dangling=true -q)'
        }
    }
}
