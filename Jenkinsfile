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
        GCP_BEARER_TOKEN= ""
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
                    sleep 600
                    //stash name: 'query-results', includes: 'query.txt'
                    
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
            agent {
                kubernetes {
                    label 'cloud-run-pod'
                    yaml gcpPodYaml
                }
            }
            steps {
                container ('gcp-sdk'){
                    GCP_BEARER_TOKEN = sh(script: 'gcloud auth print-access-token', returnStdout: true).trim()
                //unstash 'query-results'
                    //sleep 600
                    sh 'curl -X GET \ -H "Authorization: Bearer ${env.GCP_BEARER_TOKEN}" \ -o "mysql.yml" \ "https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/0/mysql.yml"'
                    //curl -H "core-flow-research: core-flow-research" -H "Authorization: Bearer ${env.GCP_BEARER_TOKEN}" 'https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/'
                    //curl --location 'https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/' \
//--header 'Authorization: Bearer ${env.GCP_BEARER_TOKEN}'
                    //sh 'gcloud auth activate-access-token "$GOOGLE_AUTH_TOKEN"'
                    //sh "gsutil cp query.txt gs://tjohns-mysql-dump/query-results/"
                }
            }
        }
    }
}
