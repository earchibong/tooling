pipeline {
    agent any

  environment 
  {
    //imagename = "ismailtest"
    //ecrurl = "https://828556645578.dkr.ecr.us-east-2.amazonaws.com"
    //ecrcredentials = "ecr:us-east-2:ecr-ismail"
    //dockerImage = ''
    PROJECT     = 'tooling'
    ECRURL      = '350100602815.dkr.ecr.eu-west-2.amazonaws.com/tooling'
    DEPLOY_TO   = 'jenkins-ecr'
  } 


  stages {

    stage("Initial cleanup") {
        steps {
        dir("${WORKSPACE}") {
            deleteDir()
        }
        }
    }

    stage('Checkout')
    {
      steps {
      checkout scmGit(
        branches: [[name: '*/jenkins-ecr']], 
        extensions: [], 
        userRemoteConfigs: [[credentialsId: 'af85cebf-4a4d-4081-90a9-b02c6a8ac15b', url: 'https://github.com/earchibong/tooling.git']]
        )
        
      }
    }

    stage('Build preparations')
    {
        steps
          {
              script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
    }   

    stage('Build') 
    {
        steps {
            echo 'Build Dockerfile....'
            script {
                //sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                sh "docker build --network=host -t $IMAGE ."
                //docker.withRegistry("https://$ECRURL"){
                //docker.image("$IMAGE").push("dev-staging-$BUILD_NUMBER")
                //}
            }
        }
    }

    stage('Test') 
    {
        steps {
            script{

                code = sh(script:'curl --location --silent --output /dev/null --write-out "%{http_code}\n" http://localhost:8085', returnStdout: true).trim()
                echo "HTTP response status code: $code"
            }           
        }
    }

    stage('Deploy') 
    {
        steps {
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                //sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("$BUILD_NUMBER-latest")
                }
            }
        }
    }
  }
 
    post
    {
        always
        {
            sh "docker rmi -f $IMAGE "
        }
    }
} //end of pipeline