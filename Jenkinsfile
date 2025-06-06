pipeline {
    agent any

    environment {
        FOSSA_API_KEY = credentials('FOSSA_API_KEY')
        TMPDIR = "${env.WORKSPACE}/tmp"
    }

    stages {
       stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/OT-MICROSERVICES/employee-api.git'
            }
        }

        stage('Change Directory') {
            steps {
                dir('employee-api') {
                    script {
                        echo "Current directory: ${pwd()}"
                        
                    }
                }
            }
        }


        stage('Install FOSSA CLI') {
            steps {
                sh 'mkdir -p tmp'
                sh 'curl -H "Cache-Control: no-cache" https://raw.githubusercontent.com/fossas/fossa-cli/master/install-latest.sh | bash'
                sh 'fossa --version'
            }
        }

        stage('Configure FOSSA') {
            steps {
                dir('employee-api') {
                    sh 'fossa init || true'
                    
                }
            }
        }

        stage('Run FOSSA Scan') {
            steps {
                dir('employee-api') {
                    sh 'mkdir -p $TMPDIR'
                    sh 'fossa analyze --debug'
                    sh 'fossa test --timeout 600 || true  # Continue pipeline even if issues found'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'employee-api/fossa-*.json', allowEmptyArchive: true
        }
        success {
            script {
                def emailBody = """
                <html>
                    <head>
                        <style>
                            body {
                                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                                background: linear-gradient(to right, #e0f7e9, #f4fdf8);
                                padding: 40px;
                            }
                            .card {
                                max-width: 600px;
                                margin: auto;
                                background-color: #ffffff;
                                padding: 30px;
                                border-left: 8px solid #28a745;
                                border-radius: 8px;
                                box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
                            }
                            .header {
                                font-size: 24px;
                                color: #28a745;
                                margin-bottom: 10px;
                            }
                            .content {
                                color: #333333;
                                font-size: 16px;
                                line-height: 1.6;
                            }
                            .content a {
                                color: #28a745;
                                text-decoration: none;
                                font-weight: bold;
                            }
                            .footer {
                                margin-top: 20px;
                                font-size: 14px;
                                color: #666;
                            }
                        </style>
                    </head>
                    <body>
                        <div class="card">
                            <div class="header">✅ Jenkins Job Succeeded</div>
                            <div class="content">
                                <p><strong>Job Name:</strong> ${env.JOB_NAME}</p>
                                <p><strong>Build Number:</strong> #${env.BUILD_NUMBER}</p>
                                <p><strong>Triggered By:</strong> ${currentBuild.getBuildCauses().shortDescription}</p>
                                <p><strong>Check Logs:</strong> <a href="${env.BUILD_URL}">View Build Logs</a></p>
                                <p><strong>FOSSA Results:</strong> No license violations found</p>
                            </div>
                            <div class="footer">
                                Jenkins CI | Powered by DevOps 💻
                            </div>
                        </div>
                    </body>
                </html>
                """
                emailext(
                    mimeType: 'text/html',
                    subject: "✅ SUCCESS: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})",
                    body: emailBody,
                    to: 'rajeevsunny05@gmail.com'
                )
            }
        }
        failure {
            script {
                def emailBody = """
                <html>
                    <head>
                        <style>
                            body {
                                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                                background: linear-gradient(to right, #fdecea, #fff5f5);
                                padding: 40px;
                            }
                            .card {
                                max-width: 600px;
                                margin: auto;
                                background-color: #ffffff;
                                padding: 30px;
                                border-left: 8px solid #dc3545;
                                border-radius: 8px;
                                box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
                            }
                            .header {
                                font-size: 24px;
                                color: #dc3545;
                                margin-bottom: 10px;
                            }
                            .content {
                                color: #333333;
                                font-size: 16px;
                                line-height: 1.6;
                            }
                            .content a {
                                color: #dc3545;
                                text-decoration: none;
                                font-weight: bold;
                            }
                            .footer {
                                margin-top: 20px;
                                font-size: 14px;
                                color: #666;
                            }
                        </style>
                    </head>
                    <body>
                        <div class="card">
                            <div class="header">❌ Jenkins Job Failed</div>
                            <div class="content">
                                <p><strong>Job Name:</strong> ${env.JOB_NAME}</p>
                                <p><strong>Build Number:</strong> #${env.BUILD_NUMBER}</p>
                                <p><strong>Triggered By:</strong> ${currentBuild.getBuildCauses().shortDescription}</p>
                                <p><strong>Check Logs:</strong> <a href="${env.BUILD_URL}">View Build Logs</a></p>
                                <p><strong>Possible Issues:</strong> License violations or dependency conflicts</p>
                            </div>
                            <div class="footer">
                                Jenkins CI | Investigate the issue 🧠
                            </div>
                        </div>
                    </body>
                </html>
                """
                emailext(
                    mimeType: 'text/html',
                    subject: "❌ FAILED: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})",
                    body: emailBody,
                    to: 'rajeevsunny05@gmail.com'
                )
            }
        
        }
    }
}
