pipeline {
  agent any

  options {
    skipDefaultCheckout(true)
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  parameters {
    string(name: 'APP_VERSION', defaultValue: '1.0.0', description: 'Versão do deploy')
  }

  environment {
    JDK = 'jdk17'
    MVN = 'maven-3.9' 
    STG_HOST = 'stg.example.internal'
    PRD_HOST = 'prd.example.internal'
    ARTIFACT = "target/hello-jenkins-${params.APP_VERSION}.jar"
  }

  tools {
    jdk "${JDK}"
    maven "${MVN}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B -q clean verify'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Package') {
      steps {
        sh "mvn -B -q -DskipTests package"
        sh "cp target/hello-jenkins-1.0.0.jar target/hello-jenkins-${params.APP_VERSION}.jar"
      }
    }

    stage('Deploy to STG') {
      steps {
        echo "Fazendo deploy da versão ${params.APP_VERSION} em STG (${env.STG_HOST})"
        sh '''
          # Exemplo placeholder:
          echo "scp ${ARTIFACT} user@${STG_HOST}:/opt/apps/hello-jenkins/"
          echo "ssh user@${STG_HOST} 'systemctl --user restart hello-jenkins.service'"
        '''
      }
    }

    stage('Approve Promotion to PRD') {
      steps {
        timeout(time: 2, unit: 'HOURS') {
          input message: "Promover versão ${params.APP_VERSION} para PRD?"
        }
      }
    }

    stage('Deploy to PRD') {
      when { beforeAgent true; expression { return currentBuild.resultIsBetterOrEqualTo('SUCCESS') } }
      steps {
        echo "Fazendo deploy da versão ${params.APP_VERSION} em PRD (${env.PRD_HOST})"
        sh '''
          # Exemplo placeholder:
          echo "scp ${ARTIFACT} user@${PRD_HOST}:/opt/apps/hello-jenkins/"
          echo "ssh user@${PRD_HOST} 'systemctl --user restart hello-jenkins.service'"
        '''
      }
    }
  }

  post {
    success {
      echo "Pipeline OK! Versão ${params.APP_VERSION} entregue em STG e (se aprovado) PRD."
    }
    failure {
      echo "Falhou. Verificar logs dos estágios."
    }
  }
}
