node{
    stage('Build'){
        docker.image('python:2-alpine').inside{
            sh 'python -m py_compile ~/Projects/submission-dicoding/simple-python-pyinstaller-app/sources/add2vals.py ~/Projects/submission-dicoding/simple-python-pyinstaller-app/sources/calc.py'

        }
    }
    try{
        stage('Test'){
            docker.image('qnib/pytest').inside{
                sh 'py.test --verbose --junit-xml test-reports/results.xml ~/Projects/submission-dicoding/simple-python-pyinstaller-app/sources/test_calc.py'
            }
        }
    } finally {
        junit 'test-reports/results.xml'
    }  
    try{
        stage('Deliver'){
            docker.image('cdrx/pyinstaller-linux:python2').inside{
                sh 'pyinstaller --onefile ~/Projects/submission-dicoding/simple-python-pyinstaller-app/sources/add2vals.py'
            }
        }
        archiveArtifacts 'dist/add2vals'
    } catch (e){
        throw e
    }


    
}