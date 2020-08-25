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
        string(name: 'no_access_users_list', description: 'List of users which must have no access to mentioned buckets.  Example: "acltestuser1 acltestuser3"')
        string(name: 'read_access_users_list', description: 'List of users which must have read access to mentioned buckets.  Example: "acltestuser2 acltestuser4"')
        string(name: 'full_access_users_list', description: 'List of users which must have full access to mentioned buckets.  Example: "acltestuser5"')
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
                    sh  '''#!/bin/bash
                        set -x
                        IFS=' ' read -r -a buckets <<< ${buckets_list}
                        IFS=' ' read -r -a no_access_users <<< ${no_access_users_list}
                        IFS=' ' read -r -a read_access_users <<< ${read_access_users_list}
                        IFS=' ' read -r -a full_access_users <<< ${full_access_users_list}
                        if [[ ${#buckets[@]} -eq 0 ]]; then
                            echo "ERROR: List of buckets is empty"
                            exit 1
                        else
                            if [[ ${#no_access_users[@]} -eq 0 ]]; then
                                echo "INFO: List of users with no access is empty"
                            else
                                no_access_principals="\\"arn:aws:iam:::user/${no_access_users[0]}\\""
                                for (( i=1; i<${#no_access_users[@]}; i++ )); do
                                    no_access_principals="${no_access_principals},\\"arn:aws:iam:::user/${no_access_users[$i]}\\""
                                done
                                echo """
                                    {
                                      \\\"Effect\\\":  \\\"Deny\\\",
                                      \\\"Principal\\\":  {
                                        \\\"AWS\\\":  [
                                          $no_access_principals
                                        ]
                                      },
                                      \\\"Action\\\": \\\"s3:*\\\",
                                      \\\"Resource\\\": [
                                        \\\"arn:aws:s3:::${bucket}\\\",
                                        \\\"arn:aws:s3:::${bucket}/*\\\"
                                      ]
                                    }
                                  """ > no_access_statement.txt
                            fi
                            if [[ ${#read_access_users[@]} -eq 0 ]]; then
                                echo "INFO: List of users with read access is empty"
                            else
                                read_access_principals="\\"arn:aws:iam:::user/${read_access_users[0]}\\""
                                for (( i=1; i<${#read_access_users[@]}; i++ )); do
                                    read_access_principals="${read_access_principals},\\"arn:aws:iam:::user/${read_access_users[$i]}\\""
                                done
                                echo """,
                                    {
                                      \\\"Effect\\\":  \\\"Allow\\\",
                                      \\\"Principal\\\":  {
                                        \\\"AWS\\\":  [
                                          $read_access_principals
                                        ]
                                      },
                                      \\\"Action\\\": [
                                        \\\"s3:ListBucket\\\",
                                        \\\"s3:ListBucketVersions\\\",
                                        \\\"s3:GetObject\\\",
                                        \\\"s3:GetObjectVersion\\\",
                                        \\\"s3:GetObjectAcl\\\",
                                        \\\"s3:GetBucketPolicy\\\",
                                        \\\"s3:GetBucketAcl\\\"
                                        ],
                                      \\\"Resource\\\": [
                                        \\\"arn:aws:s3:::${bucket}\\\",
                                        \\\"arn:aws:s3:::${bucket}/*\\\"
                                      ]
                                    }
                                  """ > read_access_statement.txt
                            fi
                            if [[ ${#full_access_users[@]} -eq 0 ]]; then
                                echo "INFO: List of users with full access is empty"
                            else
                                full_access_principals="\\"arn:aws:iam:::user/${full_access_users[0]}\\""
                                for (( i=1; i<${#full_access_users[@]}; i++ )); do
                                    full_access_principals="${full_access_principals},\\"arn:aws:iam:::user/${full_access_users[$i]}\\""
                                done
                                echo """,
                                    {
                                      \\\"Effect\\\":  \\\"Allow\\\",
                                      \\\"Principal\\\":  {
                                        \\\"AWS\\\":  [
                                          $full_access_principals
                                        ]
                                      },
                                      \\\"Action\\\": \\\"s3:*\\\",
                                      \\\"Resource\\\": [
                                        \\\"arn:aws:s3:::${bucket}\\\",
                                        \\\"arn:aws:s3:::${bucket}/*\\\"
                                      ]
                                    }
                                  """ > full_access_statement.txt
                            fi
                            for bucket in ${buckets[@]}; do
                                echo """
                                    { 
                                      \\\"Version\\\": \\\"2012-10-17\\\",
                                      \\\"Statement\\\": [
                                  """ > ${bucket}_policy.txt
                                cat no_access_statement.txt > ${bucket}_policy.txt 2>/dev/null
                                cat read_access_statement.txt > ${bucket}_policy.txt 2>/dev/null
                                cat full_access_statement.txt > ${bucket}_policy.txt 2>/dev/null
                                echo """
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
