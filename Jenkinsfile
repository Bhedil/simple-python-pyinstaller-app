node {
    stage('Checkout'){
        checkout scm
    }

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

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk mengakhiri)'
    }

    stage('Deploy') {
    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ssh-key', keyFileVariable: 'SSH_KEY')]) {
            sh """
            ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ${SSH_KEY} ${env.EC2_USER}@${env.EC2_HOST} << EOF
                # Update packages and install Docker
                sudo yum update -y && sudo yum install -y docker && sudo yum install -y git

                # Start Docker service
                sudo service docker start
                sudo usermod -a -G docker ec2-user

                # Ensure the deployment directory exists
                mkdir -p ${env.DEPLOY_DIR}

                cd ${env.DEPLOY_DIR}
                if [ -d ".git" ]; then
                    sudo git reset --hard
                    sudo git pull origin master
                else
                    sudo git clone -b master https://github.com/Bhedil/simple-python-pyinstaller-app.git .  # Replace with your repo
                fi

                # Pull the latest Python Docker image
                sudo docker pull python:3.9

                # Stop and remove the existing container if running
                sudo docker stop ${env.APP_NAME} || true
                sudo docker rm ${env.APP_NAME} || true

                # Run the application inside a persistent Docker container
                sudo docker run -d --name ${env.APP_NAME} -v ${env.DEPLOY_DIR}:/app -w /app python:3.9 bash -c "
                    pip install pyinstaller &&
                    pyinstaller --onefile sources/add2vals.py &&
                    tail -f /dev/null
                "
                # Copy the built artifact from the container's volume to EC2
                sudo cp ${env.DEPLOY_DIR}/dist/add2vals ${env.DEPLOY_DIR}/add2vals
            """

            sh """
            scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i $SSH_KEY ${env.EC2_USER}@${env.EC2_HOST}:${env.DEPLOY_DIR}/add2vals .
            """
            
            archiveArtifacts artifacts: 'add2vals', fingerprint: true
            sleep time: 1, unit: 'MINUTES'
            echo 'Deployment successfully.'
        }
    }
}
