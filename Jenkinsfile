import groovy.sql.Sql

library 'cb-days@master'
def mysqlPodYaml = libraryResource 'podtemplates/mysql.yml'
//def gcpPodYaml = libraryResource 'podtemplates/cloud-run.yml'
pipeline {
    agent none
    options { 
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout true
        preserveStashes(buildCount: 10)
    }
    environment {
        MYSQL_CREDS=credentials('mysql')
        //GOOGLE_OAUTH_TOKEN_CREDS=credentials('gcloud')
    }

    stages {
        stage('Get Params from MYSQL') {
            agent {
                kubernetes {
                    label 'mysql'
                    yaml mysqlPodYaml
                }
            }
            steps {
                container('mysql') {
                    sh(script:'''
                            mysql -u $MYSQL_CREDS_USR -p$MYSQL_CREDS_PSW -h 10.108.11.163 -P 3306 -D uno_params -e "SELECT name FROM colors;" > query.txt
                        ''')
                    //print(env.mycolors)
                    
                    sh'tail /home/jenkins/agent/workspace/mysql-test/query.txt'
                    sh'ls -l /home/jenkins/agent/workspace/mysql-test/query.txt'
                    stash name: 'query-results', includes: 'query.txt'    
                }
            }
        }
    }
}
