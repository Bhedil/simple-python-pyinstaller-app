node {
    stage('Checkout') {
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

    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside("--entrypoint='' --user root") {
            try {
            sh 'set -x'  // Enable debug mode
            sh 'echo "deb http://old-releases.ubuntu.com/ubuntu/ precise main restricted universe multiverse" > /etc/apt/sources.list'
            sh 'echo "deb http://old-releases.ubuntu.com/ubuntu/ precise-updates main restricted universe multiverse" >> /etc/apt/sources.list'
            sh 'echo "deb http://old-releases.ubuntu.com/ubuntu/ precise-security main restricted universe multiverse" >> /etc/apt/sources.list'
            sh 'apt-get update && apt-get install -y python-pip'
            sh 'pip install --index-url=https://pypi.python.org/simple/ --upgrade setuptools'
            sh 'pip install --index-url=https://pypi.python.org/simple/ pyinstaller'
            sh 'pyinstaller --version || echo "PyInstaller is still missing!"'
            sh 'ls -lah sources || echo "Sources directory is missing!"'
            sh 'pyinstaller --onefile sources/add2vals.py'
            } catch (Exception e) {
                echo "Delivery stage failed: ${e}"
            }
        }

        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}
