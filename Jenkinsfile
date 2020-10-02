pipeline{
    agent any
    environment {
        app_version = 'v2'
        rollback = 'false'
    }
    stages{

        stage('SSH Connect | Run | Test application in testing-vm'){
            steps{
                script{
                    if (env.rollback == 'false'){
                        withCredentials([file(credentialsId: 'Authentication', variable: 'AWS_EU_Key'),
                                       string(credentialsId: 'TEST_DATABASE_URI', variable: 'uri'),
                                       string(credentialsId: 'MYSQL_ROOT_PASSWORD', variable: 'pwd'),
                                       string(credentialsId: 'SECRET_KEY', variable: 'key')]){
                            sh '''
                                # SSH into testing-vm
                                ssh -tt -o "StrictHostKeyChecking=no" -i $AWS_EU_Key ubuntu@ec2-18-134-133-25.eu-west-2.compute.amazonaws.com << EOF

                                # Pull project from docker-hub
                                rm -rf cne-sfia2-project
                                git clone https://github.com/DKhan1998/cne-sfia2-project.git
                                cd cne-sfia2-project

                                # Connect to mysql instance
                                sudo apt install mysql-client-core-5.7 -y
                                mysql -h testdb.cgytirb7uezx.eu-west-2.rds.amazonaws.com -P 3306 -u admin -p$pwd


                                # Upload the database
                                source database/Create.sql;

                                exit

                                # build project using docker-compose and environment variables
                                sudo -E MYSQL_ROOT_PASSWORD=$pwd DB_PASSWORD=$pwd TEST_DATABASE_URI=$uri SECRET_KEY=$key docker-compose up -d --build

                               exit

                                >> EOF
                             '''
                        }
                    }
                }
            }
        }
        stage('Testing'){
            steps{
                script{
                    if (env.rollback == 'false'){
                        withCredentials([file(credentialsId: 'Authentication', variable: 'AWS_EU_Key'),
                                       string(credentialsId: 'TEST_DATABASE_URI', variable: 'uri'),
                                       string(credentialsId: 'MYSQL_ROOT_PASSWORD', variable: 'pwd'),
                                       string(credentialsId: 'SECRET_KEY', variable: 'key')]){
                            sh '''
                                # SSH into testing-vm
                                ssh -tt -o "StrictHostKeyChecking=no" -i $AWS_EU_Key ubuntu@ec2-18-134-133-25.eu-west-2.compute.amazonaws.com << EOF

                                sudo apt install python-pytest

#                                docker run -d -p 80:80 --name cne-sfia2-project cne-sfia2-project:${app_version}
                                pytest --durations=200 -n 11 --cov cne-sfia2-project -v frontend/tests/test_frontend.py

                                pytest --durations=200 -n 11 --cov cne-sfia2-project -v backend/tests/test_backend.py

                                exit

                                >> EOF
                            '''
                        }
                    }
                }
            }
        }
    }
}