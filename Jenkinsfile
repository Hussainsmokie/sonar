pipeline {   
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        SONARQUBE = credentials('sonar-token-id')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Hussainsmokie/sonar.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Download Sonar Report') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        PROJECT_KEY=calculator   # ✅ your actual project key
                        SONAR_HOST=$SONAR_HOST_URL
                        TOKEN=$SONAR_AUTH_TOKEN

                        echo "Fetching ALL issues for project: $PROJECT_KEY"
                        > sonar-report.txt   # clear file before writing

                        PAGE=1
                        PAGESIZE=500
                        TOTAL=1

                        while [ $(( (PAGE-1) * PAGESIZE )) -lt $TOTAL ]
                        do
                          echo "Fetching page $PAGE ..."
                          RESPONSE=$(curl -s -u $TOKEN: "$SONAR_HOST/api/issues/search?projectKeys=$PROJECT_KEY&ps=$PAGESIZE&p=$PAGE")

                          # Save raw JSON for debugging (last page only overwrites)
                          echo $RESPONSE > sonar-report.json

                          # Extract total count from JSON
                          TOTAL=$(echo $RESPONSE | jq '.total')

                          # Append issues to report file
                          echo $RESPONSE | jq -r '.issues[] | "[\(.type)] \(.severity) - \(.message) at \(.component):\(.line // "N/A")"' >> sonar-report.txt

                          PAGE=$((PAGE+1))
                        done

                        echo "✅ Sonar report saved to sonar-report.txt"
                    '''
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'sonar-report.txt', fingerprint: true
        }
    }
}
