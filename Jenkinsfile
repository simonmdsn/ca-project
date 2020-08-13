pipeline {
 agent any
 environment {
       docker_username = 'simonmdsn'
   }
 stages {
stage('Clone down') {
    steps {
        stash excludes: '.git', name: 'code'
    }
}
   stage('Run python tests') {
       agent {
           docker {
               image 'python:3'
           }
       } 
        steps {
            unstash 'code'
            sh 'ci/python-test.sh'
            stash excludes: '.git', name: 'code' //Is this step optionally
        }    
   }
   stage('Parallel') {
       parallel{
           stage('Create artifacts') {
               steps {
                   sh '''mkdir artifacts
                        tar -zcvf ./artifacts/flaskproject.tar.gz .'''
                    archiveArtifacts artifacts: 'artifacts/flaskprojects.tar.gz'
               }
           }
            stage('Build docker image'){
        agent any
        environment {
            DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
            unstash 'code' //unstash the repository code
            sh 'ci/build-docker.sh'
            stash excludes: '.git', name: 'code'
        }
   }

   }
  
   
   stage('Push docker image') {
       agent any
       environment {
           DOCKERCREDS = credentials('docker_login')
       }
       steps {
        unstash 'code' //unstash the repository code
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
        stash excludes: '.git', name: 'code'
     }

   }
 }
}