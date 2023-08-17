import groovy.sql.Sql

library 'cb-days@master'
def mysqlPodYaml = libraryResource 'podtemplates/mysql.yml'
def gcpPodYaml = libraryResource 'podtemplates/cloud-run.yml'
pipeline {
    agent none
    options { 
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout true
        preserveStashes(buildCount: 10)
    }
    environment {
        MYSQL_CREDS=credentials('mysql')
        GOOGLE_AUTH_TOKEN=credentials('gcloud-auth')
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
                    sleep 600
                    sh(script:'''
                            mysql -u $MYSQL_CREDS_USR -p$MYSQL_CREDS_PSW -h 10.108.11.163 -P 3306 -D uno_params --skip-column-names -e "SELECT JSON_OBJECT('ENV', environment, 'IMAGE', image, 'VERSION', version) FROM inventory WHERE environment = 'DEV';" > query.json
                        ''')
                    //print(env.mycolors)
                    
                    sh'tail /home/jenkins/agent/workspace/mysql-test/query.json'
                    sh'ls -l /home/jenkins/agent/workspace/mysql-test/query.json'
                    
                    stash name: 'query-results', includes: 'query.json'
                    
                }
            }
        }
        stage('Transfer to GCP Bucket') {
            agent {
                kubernetes {
                    label 'cloud-run-pod'
                    yaml gcpPodYaml
                }
            }
            steps {
                container ('gcp-sdk'){
                    script{
                        unstash 'query-results'
                        sh 'cat query.json'
                        sh "gsutil cp /home/jenkins/agent/workspace/mysql-test/query.json gs://tjohns-mysql-dump/query-results/"

                    //sh 'gcloud auth activate-access-token "$GOOGLE_AUTH_TOKEN"'
                    }
                }
            }
        }
    }
}
