pipeline {
  agent {
    docker { image 'liquibase/liquibase:4.4.2' }
  }
   parameters {
        choice choices: ['update','rollback'], description: 'Please select the opertaion', name: 'operation'
        choice choices: ['basic','state'], description: 'Please select the schema', name: 'schema'
    }
  environment {
        RDS_Mysql_CREDS=credentials('RDS-MySql')
        endpoint = "database-1.cpogtkusv37n.ap-south-1.rds.amazonaws.com:3306"
  }
  stages {
    stage('Liuibase version') {
      steps {
        sh 'liquibase --version'
      }
    }
    stage('SCM') {
      steps {
        git branch: 'main', url: 'https://github.com/Sampaati/RDS-scripts.git'
      }
    }
    stage('Liuibase update SQL and validte') {
      steps {
        echo "########## VALIDATING SQL########"            
          sh 'liquibase validate --url="jdbc:mariadb://${endpoint}/${schema}" --changeLogFile=./${schema}/changelogFile.xml --username=$RDS_Mysql_CREDS_USR --password=$RDS_Mysql_CREDS_PSW'
      }
    }
    stage('Status') {
      when {
        expression { params.operation == 'update' 
        }
      }
      steps {
        script {
          sh 'liquibase status --url="jdbc:mariadb://$(endpoint)/${schema}" --changeLogFile=./${schema}/changelogFile.xml --username=$RDS_Mysql_CREDS_USR --password=$RDS_Mysql_CREDS_PSW'
          echo "#######################################"
          echo "#########checking ####################"
          env.Proceed = input message: 'User input required',
          parameters: [choice(name: 'Do you wish to proceed', choices: 'no\nyes', description: 'Choose "yes" if you want to proceed')]
        }
      }
    }
    
    stage('Update') {
      when {
        environment name: 'Proceed', value: 'yes'
      }
      steps {
        script {
          echo "Updating db"
          sh 'liquibase update --url="jdbc:mariadb://${endpoint}/${schema}" --changeLogFile=./${schema}/changelogFile.xml --username=$RDS_Mysql_CREDS_USR --password=$RDS_Mysql_CREDS_PSW'
        }
      }
    }
    stage('Rollback') {
      when {
        expression { params.operation == 'rollback' 
      }
    }
      steps {
        script {
          echo "Rollback"
          sh 'liquibase rollback-count --count=1 --url="jdbc:mariadb://${endpoint}/${schema}" --changeLogFile=./${schema}/changelogFile.xml --username=$RDS_Mysql_CREDS_USR --password=$RDS_Mysql_CREDS_PSW'
        }
      }
    }
  }
  post {
    always {
        // can use cleanWs() which is clean work space but it affects the successive job
    //   cleanWs()
    echo 'Cleaning WOrkspace'
    sh 'rm -rf *'
    sh 'ls'
    echo 'Workspace cleaned'
    }
  }
}
