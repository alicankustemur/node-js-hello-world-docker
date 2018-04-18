env.CONTAINER_REGISTRY = "harbor.kandy.local"
env.GITHUB_CREDENTIALS_ID = "github-token"
env.DOCKER_CREDENTIALS_ID = "docker-credential"
env.GIT_REPO = "https://github.com/alicankustemur/node-js-hello-world-docker.git"
env.BRANCH = "master"
env.APP = "$CONTAINER_REGISTRY/library/nodejs"
env.WORK_DIR = "nodejs"

/* 
withCredentials([string(credentialsId: "$GITHUB_CREDENTIALS_ID",variable: 'GITHUB_TOKEN')]) {
    env.GITHUB_TOKEN = "${GITHUB_TOKEN}"
} 
*/
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
            //sh 'docker build --build-arg github_token_arg=${GITHUB_TOKEN} -t $APP .'
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
        
        sh 'docker tag $APP $APP:$BUILD_NUMBER'
        sh 'docker push $APP:$BUILD_NUMBER'
        
    }
}
