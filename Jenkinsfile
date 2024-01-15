def registry = 'https://techmahindra123.jfrog.io'
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/ShrutikaShrirao/ncplspringboot.git'
      }
    }
 stage('Pull Changes') {
      steps {
        sh 'git pull origin main'
        echo "WebHook Is Successfull Change"
      }
      }

stage ('Build') {
          steps {

            sh 'mvn clean install'    

            }
      }

   stage('Unit Test') {
      steps {
        echo '<--------------- Unit Testing started  --------------->'
        sh 'mvn surefire-report:report'
        echo '<------------- Unit Testing stopped  --------------->'
      }
    }

stage('Sonar Analysis') {
      environment {
        scannerHome = tool 'SonarQubeScanner'
      }
      steps {
        echo '<--------------- Sonar Analysis started  --------------->'
                withSonarQubeEnv('SonarQubeScanner') {
                sh "${scannerHome}/bin/sonar-scanner"
                }       
         }
      }
    
stage('Quality Gate') {
      steps {
        script {
          echo '<--------------- Quality Gate started  --------------->'
          timeout(time: 1, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
              error 'Pipeline failed due to the Quality gate issue: ${qg.status}'
            }
          }
          echo '<--------------- Quality Gate stopped  --------------->'
        }
      }
    }
stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogtoken"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/springbootApp.jar",
                                  "target": "maven-libs-snapshot",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        }    

    stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                  
                    sh 'sudo -u jenkins docker build -t myrepo .'
                }
            }
    }


    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 025692962952.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag myrepo:latest 025692962952.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
                sh 'docker push 025692962952.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
         }
        }
      }

//    stage('Deploy to EKS') {
//             steps {
//                 script {
//                     // Authenticate with the EKS cluster (ensure AWS credentials are configured)
//                     sh 'aws eks --region us-east-2 update-kubeconfig --name my-eks-cluster'
                    
//                     // Apply Kubernetes manifest files to deploy your application
//                      // sh "kubectl delete -f eks-deploy-k8s.yaml"
//                       sh "kubectl apply -f eks-deploy-k8s.yaml"
//                 }
//             }
//         }

  }
}
