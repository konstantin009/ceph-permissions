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
        string(name: 'buckets_list', description: 'List of buckets to set policies for.  Example: "bucket1 bucket2 bucket3"')
        string(name: 'read_users_list', description: 'List of users which for which read permission will be set.  Example: "acltestuser1 acltestuser3"')
        string(name: 'readwrite_users_list', description: 'List of users which for which read&write permission will be set.  Example: "acltestuser2"')
    }
    options {
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
    }
    stages {
        stage("Set bucket policies") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('aws_access_key')
                AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
            }
            steps {
                container('inbound-agent-s3cmd') {
                    sh '''#!/bin/bash
                          readarray -td' ' buckets <<<${buckets_list}; declare -p buckets
                          readarray -td' ' read_users <<<${read_users_list}; declare -p read_users
                          readarray -td' ' readwrite_users <<<${readwrite_users_list}; declare -p readwrite_users
                          if [[ ${#buckets[@]} -eq 0 ]]; then
                              echo "ERROR: buckets_list parameter is empty"
                              exit 1
                          else
                              if [[ ${#read_users[@]} -eq 0 ]]; then
                                  echo "NOTICE: No users with read permissions are specified"
                              else
                                  read_principals="\"arn:aws:iam:::user/${read_users[0]}\""
                                  for (( i=1; i<${#read_users[@]}; i++ )); do
                                      read_principals="${read_principals},\n\\\"arn:aws:iam:::user/${read_users[$i]}\\\""
                                  done
                              fi
                              if [[ ${#readwrite_users[@]} -eq 0 ]]; then
                                  echo "NOTICE: No users with read&write permissions are specified"
                              else
                                  readwrite_principals="\"arn:aws:iam:::user/${readwrite_users[0]}\""
                                  for (( i=1; i<${#readwrite_users[@]}; i++ )); do
                                      readwrite_principals="${readwrite_principals},\n\\\"arn:aws:iam:::user/${readwrite_users[$i]}\\\""
                                  done
                              fi
                              for bucket in ${buckets[@]}; do
                                  echo """
{ 
  \\\"Version\\\": \\\"2012-10-17\\\",
  \\\"Statement\\\": [
    {
      \\\"Effect\\\":  \\\"Allow\\\",
      \\\"Principal\\\":  {
        \\\"AWS\\\":  [
$(echo -e $read_principals)
        ]
      },
      \\\"Action\\\": \\\"s3:GetObject\\\",
      \\\"Resource\\\": [
        \\\"arn:aws:s3:::${bucket}\\\",
        \\\"arn:aws:s3:::${bucket}/*\\\"
      ]
    },
    {
      \\\"Effect\\\":  \\\"Allow\\\",
      \\\"Principal\\\":  {
        \\\"AWS\\\":  [
$(echo -e $readwrite_principals)
        ]
      },
      \\\"Action\\\": \\\"s3:*\\\",
      \\\"Resource\\\": [
        \\\"arn:aws:s3:::${bucket}\\\",
        \\\"arn:aws:s3:::${bucket}/*\\\"
      ]
    }
  ]
}
""" > ${bucket}_policy.txt
                                  cat ${bucket}_policy.txt
                              done
                          fi
                    '''
                }
            }
        }
    }
}
