node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
        }
        
        junit 'test-reports/results.xml'
    }

    stage('Deploy') {
    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ssh-key', keyFileVariable: 'SSH_KEY')]) {
            sh """
            ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ${SSH_KEY} ${env.EC2_USER}@${env.EC2_HOST} << "EOF"
                sudo yum update -y && sudo yum install -y docker

                sudo service docker start && sudo usermod -a -G docker ec2-user
                
                # Ensure the deployment directory exists
                mkdir -p ${env.DEPLOY_DIR}
                
                # Pull the latest Python Docker image
                sudo docker pull python:3.9
                
                # Run the application inside a persistent Docker container
                sudo docker run -d --name  ${env.APP_NAME}  -v  ${env.DEPLOY_DIR} :/app -w /app python:3.9 bash -c '
                    pip install pyinstaller &&
                    pyinstaller --onefile sources/add2vals.py
                '
            EOF
            """

            echo 'Deployment successfully.'
        }
    }
}
