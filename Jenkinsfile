pipeline {
   agent any


   options {
      disableConcurrentBuilds
}

    environments {
        IMAGE_NAME = "sandeeprawat001/multibrach-flask-app"
        GIT_USER = "sandeeprawat001"
        GIT_EMAIL = "sandeeprawat25691@gmail.com"


}
 
    stages {
      stage ('Checkout') {
         steps {
             checkout scm

}
}

      stage ('Build and push Image') {
         when { branch 'main' }
         steps {
            def IMAGE_TAg = "build-${BUILD_NUMBER}"
            
            withCredential([{usernamePassword(
                credentialsId: 'dockerhub-creds',                
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
)]} {
     sh """
     docker build -t {$IMAGE_NAME}:${IMAGE_TAG} .
     echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
     docker push ${IMAGE_NAME}:${IMAGE_TAG}
      """       
}

env.IMAGE_TAG = IMAGE_TAG
}
}

 stage('Update k8s Manifest') {
    when { branch 'main' }
    steps {
       scripts {
         withCredentials([usernamePassword(
            credentialsId: 'github-creds'
            usernamevariable: 'Git_USERNAME',
            passwordVariable: 'GIT_TOKEN'
            
)])

   sh """
   set -e
   git config user.name "$GIT_USER"
   git config user.email "$GIT_EMAIL"
   git fetch origin
   git checkout main
   git reset --hard orgin/main
   sed -i 's|image:.*| image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yml
   git add k8s/deployment.yml
   git diff --cached --quiet || git commit -m "Updated Image to ${IMAGE_TAG}"
   git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/rawatsandeep001/multibranch-repo.git main
  """
}
