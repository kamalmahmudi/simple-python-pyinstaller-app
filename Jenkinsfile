node {
    checkout scm

    docker.image('python:2-alpine').inside {
        stage('Build') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    docker.image('qnib/pytest').inside {
        try {
            stage('Test') {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } finally {
            junit 'test-reports/results.xml'
        }
    }

    stage('Manual Approval') {
        input message: 'Lanjutkan ke tahap Deploy?'
    }

    stage('Deploy') {
        def VOLUME = '$(pwd)/sources:/src'
        def IMAGE = 'cdrx/pyinstaller-linux:python2'

        dir(path: env.BUILD_ID) { 
            unstash(name: 'compiled-results')
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            archiveArtifacts "sources/dist/add2vals"

            // SCPK = submission-cicd-pipeline-kamalmahmudi
            // Need to be saved as Global properties > Environment variables
            withCredentials([sshUserPrivateKey(credentialsId: SCPK_CREDENTIALS_ID, keyFileVariable: 'keyfile', usernameVariable: 'username')]) {
                sh "scp -i ${keyfile} ./sources/dist/add2vals ${username}@${SCPK_HOST}:add2vals"
                sh "echo --- running add2vals ---"
                sh "ssh -i ${keyfile} ${username}@${SCPK_HOST} './add2vals 123 456 && ./add2vals 12.3 4.56 && sleep 1m'"
            }
        }
    }
}