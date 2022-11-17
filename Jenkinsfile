pipeline{
    agent any
    tools {
      maven 'M2_HOME'
    }
    environment {
      DOCKER_TAG = getVersion()
      BRANCH_NAME = "main"
    }
    stages{
        stage('SCM'){
            steps{
                echo "Branch is: ${env.BRANCH_NAME}"
                echo "Git branch is: ${env.GIT_BRANCH}"
                echo "Git local branch is: ${env.GIT_LOCAL_BRANCH}"
                git credentialsId: 'github', 
                    url: 'https://github.com/cmarcchen/MAVEN_WAR_HELLO_WORLD.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn -DskipTests clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t hello-world:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u ecpmarc -p ${dockerHubPwd}"
                }
                
                sh "docker push ecpmarc/hello-world:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
