def appName="my-app"
def appVersion="1.0-SNAPSHOT"

node('dockerslave'){
    tool name: 'maven', type: 'maven'
    stage('Check Prerequest'){
        withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
            sh 'env | grep PATH'
            echo "${tool 'maven'}"
            sh 'mvn -v'
        }
    }   

    stage('Get source'){
        git branch: 'master',
        url: 'https://github.com/jenkins-docs/simple-java-maven-app.git'
    }
    
    stage('Build') {
        withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
                sh 'mvn -B -DskipTests clean package'
            }
        }
            
    stage('Test') {
        withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
            sh 'mvn test'
            stash include: 'taregt/my-app-1.0-SNAPSHOT.jar', name: 'artifactStash'
        }
    }
}

node('dockerslave1'){
    tool name: 'Docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
    
    stage('Check Prerequest'){
            echo "${tool 'Docker'}"
            withEnv(["PATH=${env.PATH}:${tool 'Docker'}/bin"]){
                sh 'docker -v'
            }
    }    

    stage('Get Dockerfile'){
        git(url: 'git@github.com:topuzliev/Lecture_22.git', branch: "master", credentialsId: 'Privet')
    } 

    stage('Unstash Our Application'){
        unstash 'artifactStash'
    } 
            
    stage('Build Dockerfile'){
        withEnv(["PATH=${env.PATH}:${tool 'Docker'}/bin"]){
            sh "docker build --no-cache --build-arg APP_NAME=${appName} --build-arg APP_VERSION=${appVersion} -t myappdocker ."
            sh "docker images"
        } 
    }

    stage('Push Image'){
       withEnv(["PATH=${env.PATH}:${tool 'Docker'}/bin"]){
            withDockerRegistry(credentialsId: 'dockerhub', toolName: 'Docker', url: 'https://index.docker.io/v1/'){
                sh "docker tag myappdocker:latest topuzliev/myappdocker:latest"
                sh "docker push topuzliev/myappdocker:latest"
            }
        }
    }
}