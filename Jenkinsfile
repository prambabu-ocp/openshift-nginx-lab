
pipeline {
    agent any

    environment {
        OC = 'C:\\Tools\\OpenShift\\oc.exe'
        OPENSHIFT_SERVER = 'https://api.rm3.7wse.p1.openshiftapps.com:6443'
        OPENSHIFT_PROJECT = 'ramu-248-dev'
        MANIFEST = 'k8s\\app.yaml'
    }

    stages {
        stage('1. Checkout Git') {
            steps {
                checkout scm

                bat '''
@echo off
git --version
if not exist "%MANIFEST%" exit /b 1
echo Source files found.
'''
            }
        }

        stage('2. Validate Manifest') {
            steps {
                bat '''
@echo off
"%OC%" apply --dry-run=client -f "%MANIFEST%"
if errorlevel 1 exit /b 1
echo Client-side manifest validation passed.
'''
            }
        }

        stage('3. Connect to OpenShift') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'openshift-token',
                        variable: 'OPENSHIFT_TOKEN'
                    )
                ]) {
                    bat '''
@echo off
"%OC%" login "%OPENSHIFT_SERVER%" --token="%OPENSHIFT_TOKEN%"
if errorlevel 1 exit /b 1

"%OC%" project "%OPENSHIFT_PROJECT%"
if errorlevel 1 exit /b 1

"%OC%" whoami
if errorlevel 1 exit /b 1
'''
                }
            }
        }

        stage('4. Server-side Validation') {
            steps {
                bat '''
@echo off
"%OC%" apply --dry-run=server -f "%MANIFEST%"
if errorlevel 1 exit /b 1
echo OpenShift server-side validation passed.
'''
            }
        }

        stage('5. Deploy to OpenShift') {
            steps {
                bat '''
@echo off
"%OC%" apply -f "%MANIFEST%"
if errorlevel 1 exit /b 1

echo Waiting for Nginx rollout...
"%OC%" rollout status deployment/nginx-new --timeout=180s
if errorlevel 1 exit /b 1
'''
            }
        }

        stage('6. Verify Deployment') {
            steps {
                bat '''
@echo off
echo ===== Deployment =====
"%OC%" get deployment nginx-new
if errorlevel 1 exit /b 1

echo ===== Pods =====
"%OC%" get pods -l app=nginx-new
if errorlevel 1 exit /b 1

echo ===== Service =====
"%OC%" get service nginx-new
if errorlevel 1 exit /b 1

echo ===== Route =====
"%OC%" get route nginx-new
if errorlevel 1 exit /b 1
'''
            }
        }

        stage('7. HTTP Smoke Test') {
            steps {
                bat '''
@echo off
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ErrorActionPreference = 'Stop'; $h = & 'C:\\Tools\\OpenShift\\oc.exe' get route nginx-new -o jsonpath='{.spec.host}'; if (-not $h) { throw 'Route host is empty' }; $u = 'http://' + $h; Write-Output ('Testing ' + $u); $r = Invoke-WebRequest -UseBasicParsing -Uri $u -TimeoutSec 30; if ($r.StatusCode -ne 200) { exit 1 }; Write-Output ('HTTP ' + $r.StatusCode)"
if errorlevel 1 exit /b 1
'''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline succeeded: Nginx is deployed and the HTTP smoke test passed.'
        }
        failure {
            echo 'Pipeline failed. Review the first failed stage and its Console Output.'
        }
    }
}
