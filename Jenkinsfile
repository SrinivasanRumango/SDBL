pipeline {
    agent any

    environment {
        // Your Python + Pipenv (adjust if needed)
        PYTHON_EXE = "C:\\Users\\srini\\AppData\\Local\\Programs\\Python\\Python310\\python.exe"
        PIPENV_EXE = "C:\\Users\\srini\\AppData\\Local\\Programs\\Python\\Python310\\Scripts\\pipenv.exe"

        // Where you want the packaged zip to be copied locally
        QA_DIR   = "D:\\SBDL-deploy\\sbdl-qa"
        PROD_DIR = "D:\\SBDL-deploy\\sbdl-prod"

        ZIP_NAME = "sbdl.zip"
    }

    stages {

        stage('Check Tools') {
            steps {
                bat """
                    echo ==== JAVA ====
                    java -version

                    echo ==== PYTHON ====
                    "%PYTHON_EXE%" --version

                    echo ==== PIPENV ====
                    "%PIPENV_EXE%" --version
                """
            }
        }

        stage('Build') {
            steps {
                bat """
                    "%PIPENV_EXE%" --python "%PYTHON_EXE%"
                    "%PIPENV_EXE%" sync
                """
            }
        }

        stage('Test') {
            steps {
                bat """
                    "%PIPENV_EXE%" run pytest
                """
            }
        }

        stage('Package') {
            steps {
                powershell """
                    if (Test-Path "$env:ZIP_NAME") { Remove-Item "$env:ZIP_NAME" -Force }
                    # Change 'lib' to whatever folder/file you want to zip
                    Compress-Archive -Path "lib","log4j.properties","sbdl_main.py","sbdl_submit.sh","conf" -DestinationPath "$env:ZIP_NAME" -Force
                    Write-Host "Created $env:ZIP_NAME"
                """
            }
        }

        stage('Deploy to Local QA Folder') {
            when { branch 'release' }
            steps {
                powershell """
                    New-Item -ItemType Directory -Force -Path "$env:QA_DIR" | Out-Null
                    Copy-Item "$env:ZIP_NAME" "$env:QA_DIR\\" -Force
                    Write-Host "Copied $env:ZIP_NAME to $env:QA_DIR"
                """
            }
        }

        stage('Deploy to Local PROD Folder') {
            when { branch 'master' }
            steps {
                powershell """
                    New-Item -ItemType Directory -Force -Path "$env:PROD_DIR" | Out-Null
                    Copy-Item "$env:ZIP_NAME" "$env:PROD_DIR\\" -Force
                    Write-Host "Copied $env:ZIP_NAME to $env:PROD_DIR"
                """
            }
        }
    }
}
