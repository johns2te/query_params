import groovy.sql.Sql

library 'cb-days@master'
def mysqlPodYaml = libraryResource 'podtemplates/mysql.yml'
def gcpPodYaml = libraryResource 'podtemplates/cloud-run.yml'
pipeline {
    agent none
    environment {
        MYSQL_CREDS=credentials('mysql')
        GBUCKET_CREDS=credentials('gbucket')
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
            agent {
                kubernetes {
                    label 'gcpsql'
                    yaml gcpPodYaml
                }
            }
            steps {
                container ('gcp-sdk'){
                    unstash 'query-results'
                    sh "gsutil cp query.txt gs://tjohns-mysql-dump/query-results/"
                }
            }
        }
    }
}
