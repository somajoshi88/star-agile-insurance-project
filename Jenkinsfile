pipeline{
    agent { label  'slave1' }
   
    environment {
      DOCKER_TAG = getVersion()
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/somajoshi88/star-agile-insurance-project.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t somajoshi/healthinsurance:${DOCKER_TAG} "
            }
        }
         stage('Login2DockerHub') {

			steps {
			 withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u somajoshi -p ${dockerHubPwd}"
                }
			}
		}
        stage('DockerHub Push'){
            steps{
               
                
                sh "docker push somajoshi/healthinsurance:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server-ansible', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'samplenode3.inv', playbook: 'ansible-playbook.yml'
            }
        }
    }
}

def getVersion(){
    def currentDir = pwd()
    echo "Present Working Directory: ${currentDir}"
    def commitHash
    try {
        if (sh(script: 'git rev-parse --is-inside-work-tree', returnStdout: true).trim() == 'true') {
            commitHash = sh(label: '', returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        } else {
            commitHash = "dummyTag"
        }
    } catch (Exception e) {
        echo "Error: ${e}"
        commitHash = "dummyTag"
    }
    
    return commitHash
}
