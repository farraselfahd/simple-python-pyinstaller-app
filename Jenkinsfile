node{
    stage('Build'){
        docker.image('python:2-alpine').inside{
            checkout scm
            sh 'python -m py_compile ./sources/add2vals.py ./sources/calc.py'

        }
    }
    try{
        stage('Test'){
            docker.image('qnib/pytest').inside{
                checkout scm
                sh 'py.test --verbose --junit-xml test-reports/results.xml ./sources/test_calc.py'
            }
        }
    } finally {
        junit 'test-reports/results.xml'
    }  
    try{
        stage('Deploy'){
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
                }
        }
        archiveArtifacts 'dist/add2vals'
    } catch (e){
        throw e
    }


    
}