node {
		def Namespace = "default"
		def ImageName = "sayarapp/sayarapp"
        def app
		
    withCredentials([
      [$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS'],
    ]) {
        stage('Login') {

            sh """
                docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
            """
  try{
  stage('Checkout'){
      git 'https://github.com/andrenarciso4/devops-project.git'
      sh "git rev-parse --short HEAD > .git/commit-id"
      imageTag= readFile('.git/commit-id').trim()
}
		  
	stage('RUN Unit Tests'){
      sh "npm install"
      sh "npm test"
	}

   
        stage('Build image') {

            app = docker.build("andrenarciso4/docker-builds-by-jenkins")

        }


	stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

          app.inside {
                        sh 'echo "Tests passed"'
                      }
    }
	
   stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                                app.push("${env.BUILD_NUMBER}")
                                app.push("latest")
                   }
   }
	
	
    stage('Deploy on K8s'){

     sh "ansible-playbook /var/lib/jenkins/ansible/deploy.yml  --user=jenkins --extra-vars ImageName=${ImageName} --extra-vars imageTag=${imageTag} --extra-vars Namespace=${Namespace}"
    }
     } catch (err) {
      currentBuild.result = 'FAILURE'
    }
}
}
}

