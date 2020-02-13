pipeline {
    agent any
    stages {
        stage('Build App') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage ('Dockerize app'){
            steps{
               script{
                    echo 'Building container'
                    app= docker.build("carlosignaciom/trainschedule")
                    app.inside{
                            echo '$(curl localhost:8080)'
                    }
               }
                
            }
        }
       stage ('Push to Docker Hub'){
           when { 
              branch 'master'
           }
           steps{
              script{
                   docker.withRegistry('https://registry.hub.docker.com','docker_hub_login'){
                   app.push("${env.BUILD_NUMBER}")
                   app.push("latest")
                   }
              }
           }
       }
      stage ('Deploy to pro'){
          when { 
              branch 'master'
           }
          steps{
              input 'Are you ready to go live?'
              milestone(1)
              withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')], [usernamePassword(credentialsId: 'docker_hub_login', usernameVariable: 'DUSERNAME', passwordVariable: 'DUSERPASS')) {
              script{
                  try {
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop trainschedule\""
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm trainschedule\""
                  } catch (err) {
                            echo: 'caught error: $err'
                  }
                 
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker login -u $DUSERNAME -p $DUSERPASS\""
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name trainschedule -p 8080:8080 -d carlosignaciom/trainschedule:${env.BUILD_NUMBER}\""  
                 
                 
              }

              }
            
              
          }
          
      }
    }
}
