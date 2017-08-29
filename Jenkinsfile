#!/usr/bin/groovy

stage('Test: Compile Time Validation') {
  node {
    checkout scm
    if (env.BRANCH_NAME =~ /^feature.*$|^develop$/) {
      sh 'lwc JenkinsDemo.lw -s null' //compile infrastructure code to validate configuration
    }
    if (env.BRANCH_NAME =~ /^master$/) {
      sh 'lwc JenkinsDemo.lw -s null' //compile infrastructure code to validate configuration
    }
  }
}

stage('Test: Fugue Dry Run') {
  node {
    if (env.BRANCH_NAME =~ /^develop$/) {
      withEnv(["LUDWIG_PATH=cfg/develop"]) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'], 
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']]) {
          run_dry_run()
        }
      }
    }
    if (env.BRANCH_NAME =~ /^feature.*$/) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'],
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']]) {
          run_dry_run()
          }
    }
    if (env.BRANCH_NAME =~ /^master$/) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'],
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']]) {
          run_dry_run()
        }
    }
  }
}

stage('Deploy: Fugue Run and Update') {
  node {
    if (env.BRANCH_NAME =~ /^develop$/) {
      withEnv(["LUDWIG_PATH=cfg/develop"]) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'], 
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']])  {
          debug()
          def ret = sh(script: 'fugue status develop', returnStatus: true)
          if(ret == 0) {
            sh('fugue update develop JenkinsDemo.lw -y')
          } else {
            sh('fugue run JenkinsDemo.lw -a develop')
          }
        }
      }
    }
    if (env.BRANCH_NAME =~ /^feature\/.*$/) {
      withEnv(["LUDWIG_PATH=cfg/develop"]) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'], 
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']])  {
          def ret = sh(script: 'fugue status ${env.BRANCH_NAME}', returnStatus: true)
          if(ret == 0) {
            sh('fugue update ${env.BRANCH_NAME} JenkinsDemo.lw -y')
          } else {
            sh('fugue run JenkinsDemo.lw -a ${env.BRANCH_NAME} --account staging')
          }
        }
      }
    }

    if (env.BRANCH_NAME =~ /^master$/) {
      withEnv(["LUDWIG_PATH=cfg/production"]) {
        withCredentials([[$class: 'StringBinding', credentialsId: 'DEMO_AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'],
                         [$class: 'StringBinding', credentialsId: 'DEMO_AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY']])  {
          debug()
          def ret = sh(script: 'fugue status production', returnStatus: true)
          if(ret == 0) {
            sh('fugue update production JenkinsDemo.lw -y')
          } else {
            sh('fugue run JenkinsDemo.lw -a production')
          }
        }
      }
    }
  }
}

def debug() {
  echo(env.BRANCH_NAME)
  echo(env.LUDWIG_PATH)
  echo(env.AWS_ACCESS_KEY_ID)
  echo(env.AWS_SECRET_ACCESS_KEY)
}

// def run_validations() {
//   sh('make')
// }

def run_dry_run() {
  withCredentials([[$class: 'StringBinding', credentialsId: 'FUGUE_ROOT_USER', variable: 'FUGUE_ROOT_USER']]) {
    sh('fugue init us-east-1')
    sh("fugue user set root ${env.FUGUE_ROOT_USER}")
    sh('fugue status')
  }
}
