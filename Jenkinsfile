pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '8', artifactNumToKeepStr: '1'))
  }

  stages {
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
        sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }
    stage('Running on CentOS') {
      agent {
        label 'CentOS'
      }
      steps {
        sh "wget http://sansika773.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 7 8"
      }
    }
    stage('Running on Debian') {
      agent {
        docker 'openjdk:8u151-jre'
      }
      steps {
        sh "wget http://sansika773.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 6 7"
      }
    }
    stage('Promote to green') {
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps {
        sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
      }
    }
    stage('Promote from development branch to master') {
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
        echo "Checking out master branch"
        sh 'git pull origin'
        sh 'git checkout master'
        echo "Merging development branch changes with master branch"
        sh 'git merge development'
        echo "Pushing origin to master"
        sh 'git push origin master'
        echo "Tagging the release"
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
      }
    }
  }
}
