pipeline {

  
  agent {label "cpu"}
 
  options {

    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')

  }

   
    stages {
        stage('Set up Env') {
            steps {
                echo '#### Set up git env ####'
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'jenkins-user-for-git-2',
                    keyFileVariable: 'PRIVATE_KEY',
                    usernameVariable: 'USER')]) {
                        sh "git config --global user.name ${USER}"
                        sh "sudo cp ${PRIVATE_KEY} ~/.ssh/id_rsa && sudo chown ubuntu ~/.ssh/id_rsa "
                        sh "ssh-keyscan github.com >> ~/.ssh/known_hosts"
                }
                sh "rm -rf ./ecopia-zebra"
                sh "git clone git@github.com:sh22tan/ecopia-zebra.git"
                sh "cd ./ecopia-zebra && git checkout jenkins_try"
                echo '#### Set up docker env ####'
                withCredentials([usernamePassword(
                    credentialsId: 'jenkins-user-for-docker',
                    passwordVariable: 'PASSWD',
                    usernameVariable: 'USER')]) {
                        sh "docker login -u ${USER} -p ${PASSWD}"
                }
                echo '#### Set up aws ecopia ####'
                withCredentials([file(credentialsId: 'jenkins-user-for-aws-ecopia', variable: 'AWS_ECOPIA')]) {
                    sh "cp \$AWS_ECOPIA ./ecopia-zebra/script/.ecopia"
                }
                echo '#### Set up user public key ####'
                withCredentials([string(credentialsId: 'jenkins-user-for-debug', variable: 'PUB_KEY')]) {
                    sh "sudo echo ${PUB_KEY} >> ~/.ssh/authorized_keys"
                }
            }
        }      
        stage('Compile and Deploy') {
            steps {
                echo '#### execute auto script ####'
                sh 'cd ./ecopia-zebra/script && python3 ./auto_script.py --json=./auto_script_param.json --parallel=4'
            }
        }
    }



}
