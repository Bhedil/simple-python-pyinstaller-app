node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile $(pwd)/sources/add2vals.py $(pwd)/sources/calc.py'
        }
    }
    
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml $(pwd)/sources/test_calc.py'
        }
        
        junit 'test-reports/results.xml'
    }
    
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile $(pwd)/sources/add2vals.py'
        }
        
        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}
