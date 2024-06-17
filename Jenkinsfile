import groovy.json.JsonOutput

def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

def getBuildUser() {
    return currentBuild.rawBuild.getCause(Cause.UserIdCause)?.getUserId() ?: 'Unknown'
}

pipeline {
    agent any
    environment {
        BUILD_USER = ''
        NODE_VERSION = '14.17.0' // You can specify the version of Node.js you need
    }
    parameters {
        string(name: 'SPEC', defaultValue: 'cypress/integration/**/**', description: 'Ej: cypress/integration/pom/*.spec.js')
        choice(name: 'BROWSER', choices: ['chrome', 'edge', 'firefox'], description: 'Pick the web browser you want to use to run your scripts')
    }
    options {
        ansiColor('xterm')
    }
    stages {
        stage('Install Node.js and npm') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            if ! command -v nvm &> /dev/null
                            then
                                echo "Installing nvm..."
                                curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
                                export NVM_DIR="$HOME/.nvm"
                                [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"
                            fi
                            source ~/.nvm/nvm.sh
                            nvm install ${NODE_VERSION}
                            nvm use ${NODE_VERSION}
                            nvm alias default ${NODE_VERSION}
                            npm install -g npm@latest
                        '''
                    } else {
                        bat '''
                            if not exist "%USERPROFILE%\\.nvm" (
                                echo Installing nvm...
                                powershell -Command "Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/coreybutler/nvm-windows/master/nvm-setup.zip')"
                                powershell -Command "Start-Process -Wait nvm-setup.exe"
                            )
                            nvm install ${NODE_VERSION}
                            nvm use ${NODE_VERSION}
                            nvm alias default ${NODE_VERSION}
                            npm install -g npm@latest
                        '''
                    }
                }
            }
        }
        stage('Build') {
            steps {
                echo "Building the application"
            }
        }
        stage('Testing') {
            steps {
                sh 'npm install'
                sh "npx cypress run --browser ${params.BROWSER} --spec ${params.SPEC}"
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying"
            }
        }
    }
    post {
        always {
            script {
                BUILD_USER = getBuildUser()
            }
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'cypress/report',
                reportFiles: 'index.html',
                reportName: 'HTML Report'
            ])
            deleteDir()
        }
    }
}
