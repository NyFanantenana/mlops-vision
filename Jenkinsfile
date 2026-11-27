pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
    }

    parameters {
        string(
            name: 'DEPLOY_PORT',
            defaultValue: '8081',
            description: 'Port HTTP du site Nginx'
        )
    }

    environment {
        APP_NAME = 'mlops-vision'
        CONTAINER_NAME = 'mlops-vision-web'
        IMAGE_NAME = 'nginx:alpine'
    }

    stages {

        stage('1 - Checkout') {
            steps {
                echo 'Recuperation du code source...'
                checkout scm

                sh '''
                    echo "Fichiers du projet :"
                    ls -la
                '''
            }
        }

        stage('2 - Verification structure') {
            steps {
                sh '''
                    set -eu

                    test -f index.html
                    test -f css/style.css

                    echo "Structure du projet valide."
                '''
            }
        }

        stage('3 - Validation HTML') {
            steps {
                sh '''
                    set -eu

                    test -s index.html

                    grep -qi '<!DOCTYPE html' index.html
                    grep -qi '<html' index.html
                    grep -qi '</html>' index.html
                    grep -qi '<head>' index.html
                    grep -qi '<body>' index.html

                    echo "Verification HTML reussie."
                '''
            }
        }

       
        
        
        
        
        /*stage('4 - Validation CSS') {
            steps {
                sh '''
                    set -eu

                    test -s css/style.css

                    python3 - <<'PY'
from pathlib import Path

css = Path("css/style.css").read_text()

if css.count("{") != css.count("}"):
    raise SystemExit("Erreur : accolades CSS non equilibrees")

if not css.strip():
    raise SystemExit("Erreur : fichier CSS vide")

print("Verification CSS reussie.")
PY
                '''
            }
        }
*/

stage('4 - Validation CSS') {
            steps {
                sh '''
                    set -eu

                    test -s css/style.css

                    open_braces=$(tr -cd '{' < css/style.css | wc -c)
                    close_braces=$(tr -cd '}' < css/style.css | wc -c)

                    test "$open_braces" -gt 0
                    test "$open_braces" -eq "$close_braces"

                    echo "Verification CSS reussie."
                '''
            }
        }



        
        stage('5 - Analyse du projet') {
            steps {
                sh '''
                    echo "Nombre de fichiers :"
                    find . -type f \
                        ! -path './.git/*' \
                        ! -path './.gitignore' | wc -l

                    echo "Taille du projet :"
                    du -sh .
                '''
            }
        }

        stage('6 - Archivage') {
            steps {
                archiveArtifacts(
                    artifacts: 'index.html,css/**,README.md',
                    allowEmptyArchive: true,
                    fingerprint: true
                )
            }
        }

        stage('7 - Deploiement Nginx') {
            steps {
                sh '''
                    set -eu

                    command -v docker

                    echo "Telechargement de Nginx..."
                    docker pull "$IMAGE_NAME"

                    echo "Suppression de l'ancienne version..."
                    docker rm -f "$CONTAINER_NAME" || true

                    echo "Demarrage du serveur Nginx..."
                    docker run -d \
                        --name "$CONTAINER_NAME" \
                        --restart unless-stopped \
                        -p "${DEPLOY_PORT}:80" \
                        "$IMAGE_NAME"

                    echo "Copie des fichiers HTML et CSS..."
                    docker cp index.html \
                        "$CONTAINER_NAME:/usr/share/nginx/html/index.html"

                    docker cp css \
                        "$CONTAINER_NAME:/usr/share/nginx/html/css"

                    echo "Deploiement termine."
                '''
            }
        }

        stage('8 - Test HTTP') {
            steps {
                sh '''
                    set -eu

                    echo "Verification du serveur Nginx..."

                    docker exec "$CONTAINER_NAME" \
                        wget -q -O /dev/null http://127.0.0.1/

                    echo "HTTP : application accessible dans Nginx."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS : Pipeline MLOps Vision terminee avec succes.'
            echo 'Application deployee sur Nginx.'
        }

        failure {
            echo 'FAILURE : Une erreur est survenue dans la pipeline.'
        }

        always {
            echo 'Fin de l execution Jenkins.'
        }
    }
}
