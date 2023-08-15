import groovy.sql.Sql

library 'cb-days@master'
def testPodYaml = libraryResource 'podtemplates/mysql.yml'
pipeline {
    agent none
    environment {
        MYSQL_CREDS=credentials('mysql')
        GBUCKET_CREDS=credentials('gbucket')
    }

    stages {
        stage('get pods') {
            agent {
                kubernetes {
                    label 'mysql'
                    yaml testPodYaml
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
        stage('Transfer to Master') {
            steps {
                unstash 'query-results'
                sh "gcloud auth activate-service-account --key-file=$GBUCKET_CREDS_PSW"
                sh "gsutil cp /home/jenkins/agent/workspace/mysql-test/query.txt gs://tjohns-mysql-dump/query-results/
            }
        }
    }
}
