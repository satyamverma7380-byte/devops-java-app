pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "satyamverma7380/devops-java-app"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        GITHUB_CREDENTIALS = "github-credentials"
    }

    stages {

        stage('Git Clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/satyamverma7380-byte/devops-java-app.git'
            }
        }

        stage('Build Application') {
            steps {
                echo 'Building Java application using Docker multi-stage build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                sh '''
                    sed -i -E \
                    "s|(^[[:space:]]*image:[[:space:]]*).*$|\\1${DOCKER_IMAGE}:${BUILD_NUMBER}|" \
                    k8s/deployment.yaml

                    git config user.name "Jenkins"
                    git config user.email "jenkins@users.noreply.github.com"

                    git add k8s/deployment.yaml

                    if ! git diff --cached --quiet; then
                        git commit -m "Update Docker image to ${BUILD_NUMBER}"
                    else
                        echo "No manifest changes detected"
                    fi
                '''
            }
        }

        stage('Push Changes to GitHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${GITHUB_CREDENTIALS}",
                        usernameVariable: 'GITHUB_USER',
                        passwordVariable: 'GITHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu

                        cat > "$WORKSPACE/.git-askpass.sh" <<'EOF'
#!/bin/sh
case "$1" in
    *Username*) printf '%s\\n' "$GITHUB_USER" ;;
    *Password*) printf '%s\\n' "$GITHUB_TOKEN" ;;
esac
EOF

                        chmod 700 "$WORKSPACE/.git-askpass.sh"

                        trap 'rm -f "$WORKSPACE/.git-askpass.sh"' EXIT

                        GIT_ASKPASS="$WORKSPACE/.git-askpass.sh" \
                        GIT_TERMINAL_PROMPT=0 \
                        git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "Congratulations on your wedding Aditi (My Dear's friend)!",
                body: """ Wishing you Aditi a lifetime of love, happiness, understanding, and togetherness.
                May your new journey be filled with beautiful memories and endless blessings.
                
                Best wishes to the newly married couple!
                Enjoy the day !!
                
                Regard
                Satyam Verma """,
                to: "mskrishivani1@gmail.com,"
            )
        }

        failure {
            emailext (
                subject: "build failed",
                body: "Build not completed successfully",
                to: "satyamverma7380@gmail.com"
            )
        }
    }
}
