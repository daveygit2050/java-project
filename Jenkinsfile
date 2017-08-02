pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }

  stages {
    stage('Say Hello') {
      agent any

      steps {
        sayHello 'awesome student'
      }
    }
    stage('Git Information') {
      agent any

      steps {
        echo "My Branch Name: ${env.BRANCH_NAME}"
        script  {
          def myLib = new daveygit.git.gitStuff();
          echo "My Commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
        }
      }
    }
    stage('Unit Tests') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }
    stage('build') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }
    stage('deploy') {
      agent {
        label 'apache'
      }
      steps {
        sh "if ![ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
        sh "cp dist/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }
    stage("Running on CentOS") {
      agent {
        label 'CentOS'
      }
      steps {
        sh "wget http://daveyrand1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage("Test on Debian") {
      agent {
        docker 'openjdk:8u121-jre'
      }
      steps {
        sh "cd ~"
        sh "wget http://daveyrand1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage("Promote to Green") {
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps {
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
      }
    }
    stage('Promote development branch to master') {
      agent {
        label 'apache'
      }
      when {
        branch 'development'
      }
      steps {
        echo "Stashing any local changes"
        sh 'git stash'
        echo "Checking out development branch"
        sh 'git checkout development'
        sh 'git pull origin'
        echo "Checkout out master branch"
        sh 'git checkout master'
        echo "Merging development into master"
        sh 'git merge development'
        echo "Pushing to origin master"
        sh 'git push origin master'
        echo 'Tagging release'
        sh "git tag rectangle-${MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${MAJOR_VERSION}.${env.BUILD_NUMBER}"
      }
      post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development promoted to Master!",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development promoted to Master!":</p>
            <p>Check console output at &QUOT;<a href='${BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "dave@randall-it.uk"
          )
        }
      }
    }
  }
  post {
    failure {
      emailext(
        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
        <p>Check console output at &QUOT;<a href='${BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "dave@randall-it.uk"
      )
    }
  }
}
