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
        stage('Set Bearer Token') {
            agent any
            steps {
                script {
                    sh 'whoami'
                }
            }
        }
        stage('Get Params from MYSQL') {
            agent {
                kubernetes {
                    label 'mysql'
                    yaml mysqlPodYaml
                }
            }
            steps {
                container('mysql') {
                    sleep 6000
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
                        def bearerToken = "ya29.a0AfB_byB0fC2IOCwjQ9CbX_tfoo9uKhTX0g4cvnzZ-0hzvybvk4Ed5Q5dvilsHFUOFQjVTLoW4iC7XLkI-o7j5TJIRApNVXyZJK1naQwbVcnet1-_9UVPc-Vf-EgM-mw2Vz7k1IESTr_m5KoSchSvYmCZHQCN6m3JzRny0wHFrYbjr8QuoEWw7ruLQ3u5aAlL0ozg2ags0jAtgEpKskO9CqEdI6Ni_PzW0rUiGI-D4XEw7LF6EwjczDXM-JWPOL4LhYpQSyok1GyfL__MUYcKECK4a7Bv1eunxqzKYv_q-5uBvE25eAVgHspN7PaHDAqGAM1lYLl_IAZ1SzaboBTZqvsqelBmummkTM3gTfjcA_TfvPW_Nfoot6ohdAgTDIuK8eEg6yuZ64ex2BISaOM6WPvoLBAaCgYKAccSARISFQHsvYlsdY2SICnBdT7rEqhEIWvnGA0418"
                        //sleep 600
                        //sh 'curl -X GET -H "Authorization: ${bearer}" "https://www.googleapis.com/storage/v1/b/tjohns-mysql-dump/"'
                        sh 'curl -X POST -T /home/jenkins/agent/workspace/mysql-test/query.json -H "Authorization: Bearer ${bearerToken}" -H "Content-Type: application/json" "https://storage.googleapis.com/upload/storage/v1/b/tjohns-mysql-dump/o?uploadType=media&name=query.json"'
                        //sh 'curl -X PUT -H "Authorization: Bearer ${bearerToken}" -T "/home/jenkins/agent/workspace/mysql-test/query.json" "https://storage.googleapis.com/storage/v1/b/tjohns-mysql-dump/query.json"'


                    //sh 'gcloud auth activate-access-token "$GOOGLE_AUTH_TOKEN"'
                    //sh "gsutil cp query.txt gs://tjohns-mysql-dump/query-results/"
                    }
                }
            }
        }
    }
}
