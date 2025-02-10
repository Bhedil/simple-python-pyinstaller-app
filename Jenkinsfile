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
            docker run --rm -v "$(pwd):/src/" ${BASE_IMAGE}-${TARGET_OS}:${IMAGE_TAG} "pyinstaller file-creator.py"
            } catch (Exception e) {
                echo "Delivery stage failed: ${e}"
            }
        }

        archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
    }
}
