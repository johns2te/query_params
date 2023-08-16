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
        //GOOGLE_AUTH_TOKEN=credentials('gcloud-auth')
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
                            mysql -u $MYSQL_CREDS_USR -p$MYSQL_CREDS_PSW -h 10.108.11.163 -P 3306 -D uno_params -e "SELECT name FROM colors;" > query.json
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
                        def bearerToken = sh(script: 'gcloud auth print-access-token', returnStdout: true).trim()
                        //sleep 600
                        sh 'cat query.json'
                        sleep 600
                        sh 'ls -l query.json'
                        //sh 'curl -X GET -H "Authorization: Bearer ${bearer_token}" -o "mysql.yml" "https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/mysql.yml"'
                        sh 'curl -X PUT -H "Authorization: Bearer ${bearerToken}" -T "/home/jenkins/agent/workspace/mysql-test/query.json" "https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/query.json"'


                    //sh 'gcloud auth activate-access-token "$GOOGLE_AUTH_TOKEN"'
                    //sh "gsutil cp query.txt gs://tjohns-mysql-dump/query-results/"
                    }
                }
            }
        }
    }
}
