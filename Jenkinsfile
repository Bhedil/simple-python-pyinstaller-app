node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile add2vals.py ./calc.py'
        }
    }
    
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml ./test_calc.py'
        }
        
        junit 'test-reports/results.xml'
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile ./add2vals.py'
        }
        
        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}
