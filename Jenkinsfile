node {
    stage('Fix Git Safe Directory') {
        try {
            sh 'git config --global --add safe.directory /home/Documents/dicoding/simple-python-pyinstaller-app'
            echo "Git safe directory configured successfully."
        } catch (Exception e) {
            echo "Error setting Git safe directory: ${e}"
        }
    }

    stage('Verify Sources') {
        try {
            sh 'ls -lah /home/Documents/dicoding/simple-python-pyinstaller-app/sources'
        } catch (Exception e) {
            echo "Error: Sources directory is missing!"
        }
    }

    stage('Check Working Directory') {
        try {
            sh 'pwd'
            sh 'ls -lah'
        } catch (Exception e) {
            echo "Error: Working directory issue!"
        }
    }

    stage('Build') {
        docker.image('python:2-alpine').inside {
            try {
                sh 'cd /home/Documents/dicoding/simple-python-pyinstaller-app && python -m py_compile sources/add2vals.py sources/calc.py'
            } catch (Exception e) {
                echo "Build failed: ${e}"
            }
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml ./sources/test_calc.py'
        }
        
        junit 'test-reports/results.xml'
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile ./sources/add2vals.py'
        }
        
        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}