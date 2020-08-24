pipeline {
    agent {
        kubernetes {
yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: inbound-agent-s3cmd
    image: 'konstantin009/jenkins.inbound-agent4.3-4:1.0.0-s3cmd'
    command:
    - cat
    tty: true
"""
        }
    }
    parameters {
        string(name: 'aws_host', defaultValue: 'rookceph.datalake-dev.bss.net.sap:50070', description: 'Storage endpoint')
        string(name: 'bucket', defaultValue: 'di3', description: 'Storage bucket name')
        string(name: 'objects_list', description: 'List of objects (files/folders) to set permissions for.  Example: "/acltest/file1 /acltest/folder2"')
        string(name: 'read_users_list', description: 'List of users which for which read permission will be set.  Example: "acltestuser1@sap.com acltestuser3@sap.com"')
        string(name: 'readwrite_users_list', description: 'List of users which for which read&write permission will be set.  Example: "acltestuser2@sap.com"')
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
    }
    stages {
//        stage("Environment preparation") {
//            steps {
//                sh "apt-get install -y s3cmd"
//            }
//        }
        stage("Set users permissions") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws_access_key')
                AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
            }
            steps {
                sh '''
                          objects=("$(echo ${objects_list} | sed 's/,/ /g'"))
                          echo ${#objects[@]}
                '''
            }
        }
    }
}
