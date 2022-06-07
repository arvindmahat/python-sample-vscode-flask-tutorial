pipeline {
    agent { label 'build_java_11' }
    stages {

        stage('SourceCode') {
            steps {
                git branch: 'master', url: 'https://github.com/arvindmahat/python-sample-vscode-flask-tutorial.git'
            }
        }

        stage('Install dependencies') {
            steps {
                sh  'python3 -m pip install --upgrade pip setuptools wheel'
            }
        }
        stage('Install requirements') {
            steps {
                sh  'pip install -r requirements.txt'
            }
        }
        stage('Unit tests') {
            steps {
                sh  'python3 -m pytest --verbose --junit-xml test-reports/results.xml'
            }
        }
        stage('Sonarqube') {
            environment {
                scannerHome = tool 'Sonar_scanner'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage ('Artifactory Configuration') {
            steps {
                rtPipResolver (
                    id: "PIP_RESOLVER",
                    serverId: "Jfrog_Instance",
                    repo: "mypythonproject-pypi-local"
                )
            }
        }
        stage ('pip install') {
            steps {
                rtPipInstall (
                    resolverId: "PIP_RESOLVER",
                    args: "-r requirements.txt"
                )
            }
        }
        stage ('package') {
            steps {
                sh "python3 setup.py sdist upload -r local"
                sh " python3 setup.py bdist_wheel upload -r local"
            }
            
        }
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "Jfrog_Instance"
                )
            }
        }
        stage('archiveArtifacts and test results') {
            steps {
                archiveArtifacts artifacts: '__pycache__/test_test1.cpython-38-pytest-7.1.2.pyc', followSymlinks: false
                junit 'test-reports/results.xml'
            }
        }
    }
}
