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
                    
                    sh(script:'''
                            mysql -u $MYSQL_CREDS_USR -p$MYSQL_CREDS_PSW -h 10.108.11.163 -P 3306 -D uno_params -e "SELECT name FROM colors;" > query.txt
                        ''')
                    //print(env.mycolors)
                    
                    sh'tail /home/jenkins/agent/workspace/mysql-test/query.txt'
                    sh'ls -l /home/jenkins/agent/workspace/mysql-test/query.txt'
                    stash name: 'query-results', includes: 'query.txt'
                    
                    //script {
                       //sh "gcloud auth activate-service-account --key-file=$GBUCKET_CREDS_PSW"
                        //sh "gsutil cp /home/jenkins/agent/workspace/mysql-test/query.txt gs://tjohns-mysql-dump/query-results/'"

                        /*def credentialsId = env.GBUCKET_CREDS_PSW // Set your credentials ID
                        def bucketName = 'tjohns-mysql-dump'
                        def sourceFilePath = '/home/jenkins/agent/workspace/mysql-test/query.txt'
                        def destinationPath = '/query-results/'
    
                        googleStorageUpload(
                            bucket: bucketName,
                            credentialsId: credentialsId,
                            source: sourceFilePath,
                            destination: destinationPath
                        )
                        */
                        
                //}
                    
                }
            }
        }
        stage('Transfer to GCP Bucket') {
            agent any
            steps {
                //container ('gcp-sdk'){
                unstash 'query-results'
                sleep 600
                curl -X POST \-H "Authorization: Bearer ${'gcloud auth print-access-token'}" \-H "Content-Type: text/plain" \-T "${'/query.txt'}" \"${'https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/'}"
                    //sh 'gcloud auth activate-access-token "$GOOGLE_AUTH_TOKEN"'
                    //sh "gsutil cp query.txt gs://tjohns-mysql-dump/query-results/"
                }
            }
        }
    }
}
