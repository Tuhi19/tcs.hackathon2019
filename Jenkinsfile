node {
   stage('Preparation') { 
      git url: 'https://github.com/Tuhi19/tcs.hackathon2019.git', branch: 'master'          
   }
   stage('installation'){
	sh 'cd /root/work/playbooks'
	sh 'ansible-playbook master_playbook.yml'
   
   }
   stage('Build') {
      sh 'mvn clean install'
   }
   
   stage('SonarQube analysis') {
    withSonarQubeEnv(credentialsId: 'sonarqube2', installationName: 'devops2sonarqube') { 
      sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar'
    }
  }
   stage('Tag Creation') {
      withCredentials([usernamePassword(credentialsId: 'github',passwordVariable: 'GIT_PASSWORD',usernameVariable: 'GIT_USERNAME')]){
         sh "git tag 1.$BUILD_NUMBER"
         sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Tuhi19/tcs.hackathon2019.git 1.$BUILD_NUMBER"
      }
   }
   
   stage('Building image'){
       
       docker.withRegistry("http://35.225.187.141:8123/service/rest/v1","sonatypenexus"){
           def customImage = docker.build("tcsdevopsgroup2/jenkins-pipeline-hackathon:1.${BUILD_NUMBER}")
            customImage.push()
       }
   }
   stage('Pull and Run image'){
       
       docker.withRegistry("http://35.225.187.141:8123/service/rest/v1","sonatypenexus"){
           def dockerImage = docker.image("tcsdevopsgroup2/jenkins-pipeline-hackathon:1.${BUILD_NUMBER}").withRun('-p 9080:8761','',{
                    sh 'sleep 1m'
               }
            )
       }
   }
   stage('email notification'){
       emailext (
          subject: "STATUS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
          body: """<p>STATUS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
          recipientProviders: [[$class: 'DevelopersRecipientProvider']]
        )
   }
}
