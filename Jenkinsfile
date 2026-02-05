pipeline {
    agent any

    options { skipDefaultCheckout() }

    stages {
        stage('Get code') {
            steps {
                echo 'Obtenemos el código fuente'
                checkout scm
                echo "La referencia completa de git es: ${env.GIT_BRANCH}"
                stash name:'codigo', includes:'**'
                sh 'ls -la'
            }
        }
        stage('Static test') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    unstash name:'codigo'
                    sh '''
                        python3 -m flake8 --exit-zero --format=pylint todo_list-aws/src>flake8.out
                        '''
                    sh '''
                        python3 -m bandit -r todo_list-aws -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {severity}: {msg}" || true
                        '''
                    recordIssues qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]], tools: [flake8(name: 'Flake8',pattern: 'flake8.out'),pyLint(name: 'Bandit',pattern: 'bandit.out')]
                }
            }
            post { always { cleanWs() } }   
        }
    }
}