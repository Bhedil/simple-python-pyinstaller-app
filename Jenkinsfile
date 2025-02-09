node {
    stage('Fix Git Safe Directory') {
        sh 'git config --global --add safe.directory /var/jenkins_home/workspace/simple-python-pyinstaller-app'
    }

    stage('Checkout') {
        steps {
            checkout scm
            sh 'ls -lah'
            sh 'ls -lah sources'
        }
    }

    stage('Debug Inside Container') {
        docker.image('python:2-alpine').inside {
            sh 'pwd'
            sh 'ls -lah'
            sh 'ls -lah sources'
        }
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
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile sources/add2vals.py'
        }

        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}
