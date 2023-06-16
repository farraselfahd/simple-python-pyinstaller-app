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
        input message: 'Lanjutkan ke tahap Deploy ?'
    } finally {
        junit 'test-reports/results.xml'
    }  
    try{
        def remote = [:]
        remote.name = 'pyinstaller-app'
        remote.host = '54.151.250.226'
        remote.user = 'ubuntu'
        remote.password = ''
        remote.identityFile = 'jenkins-ec2-connection.pem'
        remote.allowAnyHosts = true

        withEnv(['VOLUME=$(pwd)/sources:/src','IMAGE=cdrx/pyinstaller-linux:python2']){
            stage('Deploy'){
            dir(path: env.BUILD_ID) { 
                    checkout scm
                    // unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
                }
            }
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"

            // deploy to ec2
            sshPut remote: remote, from: "${env.BUILD_ID}/sources/dist/add2vals", into: '.'
            sshCommand remote: remote, command: "chmod +x add2vals"
            sshScript remote: remote, script: "./add2vals 20 6"
            sleep 60
        }
        
    } catch (e){
        throw e
    }


    
}