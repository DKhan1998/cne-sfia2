pipeline{
    agent any
    environment {
        app_version = 'v3'
        rollback = 'false'
    }
    stages{
        stage('Build Containers'){
            steps{
                script{
                    if (env.rollback == 'false'){
                          withCredentials([file(credentialsId: 'Private-key', variable: 'key')]){
                            load "Ansible/.envvars/tf_db.groovy"
                            load "Ansible/.envvars/tf_ansible.groovy"
                            sh """
                                ssh -tt -o "StrictHostKeyChecking=no" -i '${key}' ${env.jenkins_user} << EOF

                                git clone https://github.com/DKhan1998/cne-sfia2-project.git
                                cd cne-sfia2-project

                                # Export variables to build project
                                export MYSQL_ROOT_PASSWORD=${env.MYSQL_ROOT_PASSWORD}
                                export DB_PASSWORD=${env.DB_PASSWORD}
                                export TEST_DATABASE_URI=${env.TEST_DATABASE_URI}
                                export DATABASE_URI=${env.DATABASE_URI}
                                export SECRET_KEY=${env.SECRET_KEY}

                                # build project using docker-compose and environment variables
                                sudo -E MYSQL_ROOT_PASSWORD=${env.MYSQL_ROOT_PASSWORD} DB_PASSWORD=${env.DB_PASSWORD} TEST_DATABASE_URI=${env.TEST_DATABASE_URI} SECRET_KEY=${env.SECRET_KEY} docker-compose up -d --build

                                exit

                                >> EOF
                             """
                         }
                    }
                }
            }
        }
//         stage('SSH Connect | Run | Test application in testing-vm'){
//             steps{
//                 script{
//                     if (env.rollback == 'false'){
//                         withCredentials([file(credentialsId: 'Private-key', variable: 'key')]){
//                             load "./Ansible/.envvars/tf_ansible.groovy"
//                             load "./Ansible/.envvars/tf_db.groovy"
//                             sh """
//                                 # SSH into testing-vm
//                                 ssh -tt -o "StrictHostKeyChecking=no" -i '$key' ${env.jenkins_user} << EOF
//
//                                 sudo docker-compose pull
//
//                                 sudo docker-compose run -d
//
//                                 exit
//
//                                 >> EOF
//                              """
//                         }
//                     }
//                 }
//             }
//         }
        stage('Testing'){
            steps{
                script{
                    if (env.rollback == 'false'){
                        withCredentials([file(credentialsId: 'Private-key', variable: 'key')]){
                        load "./Ansible/.envvars/tf_ansible.groovy"
                        load "./Ansible/.envvars/tf_db.groovy"
                        sh """
                            # SSH into testing-vm
                            ssh -tt -o "StrictHostKeyChecking=no" -i '${key}' ${env.testvm_user} << EOF

                            cd cne-sfia2-project

                            # Export variables to build project
                            export MYSQL_ROOT_PASSWORD=${env.MYSQL_ROOT_PASSWORD}
                            export DB_PASSWORD=${env.DB_PASSWORD}
                            export TEST_DATABASE_URI=${env.TEST_DATABASE_URI}
                            export DATABASE_URI=${env.DATABASE_URI}
                            export SECRET_KEY=${env.SECRET_KEY}

                            sudo -E TEST_DATABASE_URI=${env.TEST_DATABASE_URI} SECRET_KEY=${env.SECRET_KEY} docker exec front pytest  --cov-report term --cov=application
                            sudo -E TEST_DATABASE_URI=${env.TEST_DATABASE_URI} SECRET_KEY=${env.SECRET_KEY} docker exec back pytest  --cov-report term --cov=application

                            exit

                            >> EOF
                        """
                        }
                    }
                }
            }
        }
    }
}
