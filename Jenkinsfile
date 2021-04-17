pipeline {
    agent any
    tools{
        maven 'maven'
        jdk 'jdk15'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage("build & SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('sonarQube') {
                sh 'mvn clean package sonar:sonar'
              }
            }
          }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
            post{
                success{
                    echo "========Deploying executed successfully========"
                    mail bcc: '', body: 'deploying is sucesfull', cc: '', from: '', replyTo: '', subject: 'deployed', to: 'vaishnavidr123@gmail.com'
                }
                failure{
                 echo "========Deploying failed========"
                 mail bcc: '', body: 'deploying failed', cc: '', from: '', replyTo: '', subject: 'deployed', to: 'vaishnavidr123@gmail.com'
                }
            }
        }
      stage('collect artifact'){
           steps{
               archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
           }
        }
           stage('deploy to artifactory')
           {
              steps{
                    rtUpload (
                        serverId: 'ARTIFACTORY_SERVER',
                            spec: '''{
                                "files": [
                                {
                                    "pattern": "target/*.jar",
                                    "target": "art-doc2-dev-loc/sample/"
                                }
                                ]
                            }''',
                buildName: 'holyFrog',
                buildNumber: '42'
                )
            }
           }
         stage('download from artifactory')
           {
              steps{
                    rtDownload (
                    serverId: 'ARTIFACTORY_SERVER',
                    spec: '''{
                    "files": [
                         {
                             "pattern": "art-doc2-dev-loc/sample/",
                             "target": "src/"
                        }
                     ]
                     }''',
 
                    buildName: 'holyFrog',
                    buildNumber: '42'
                )
              }
           }
        
         stage('upload to ec2')
           {
              steps{
          sshagent(['513fbce6-0f7a-4bc0-8cd9-66f09ab7ab56']){
                   sh 'ssh -o StrictHostKeyChecking=no ubuntu@18.221.112.248 pwd'
                   sh 'scp -r C:/Windows/System32/config/systemprofile/AppData/Local/Jenkins/.jenkins/workspace/jfrog-artifactory-pipeline/target/*.jar ubuntu@18.221.112.248:/home/ubuntu/amyrtifacts'
                     }
              }
           }
         stage('upload to s3')
           {
              steps{
        withAWS(region:'us-west-2',credentials:'eaaf81ec-205e-40de-ad8f-a21281406850') {
                    s3Upload(file:'target/*.jar', bucket:'vaishnavidrbucket', path:'artifacts/')
        }
              }
        }
    }
}
