podTemplate(
    envVars: [
        envVar(key: 'PROJECT_NAME', value: "cxp"),
        envVar(key: 'SERVICE_NAME', value: "docker-reactjs-demo"),
        envVar(key: 'SERVICE_REPO_URL', value: "https://github.com/quangno129/docker-reactjs-demo.git"),
        envVar(key: 'VERSION', value: "1.0"),
        envVar(key: 'ECR_URL', value: "xxxxxxxxxxxx"),
        envVar(key: 'NEED_TO_OVERRIDE_CONFIG', value: "false"),
        envVar(key: 'NEED_ENV_FILE', value: "true"),
        envVar(key: 'TELEGRAM_NOTIFICATION_ENABLED', value: "true"),
        envVar(key: 'TELEGRAM_CHAT_ID', value: "-xxxxxxxxxxxx"),
        envVar(key: 'FILE_CONFIG', value: ".env"),
        envVar(key: 'PATH_TO_CONFIG', value: "./"),
        envVar(key: 'TELEGRAM_BOT_TOKEN', value: "xxxxxxxxxxxx"),
    ],
    yaml: """
        apiVersion: "v1"
        kind: "Pod"
        spec:
            tolerations:
            - key: "dedicated"
              operator: "Equal"
              value: "JenkinsAgentGroup"
              effect: "NoSchedule"
            - key: "dedicated"
              operator: "Equal"
              value: "JenkinsAgentGroup"
              effect: "NoExecute"
    """,
    namespace: 'jenkins',
    nodeSelector: 'group=JenkinsAgentGroup',
    containers: [
        containerTemplate(name: 'kaniko', image: 'gcr.io/kaniko-project/executor:v1.18.0-debug', ttyEnabled: true, privileged: true, command: 'cat'),
        containerTemplate(name: 'sonar-scanner', image: 'sonarsource/sonar-scanner-cli:5.0.1', ttyEnabled: true, privileged: true, command: 'cat'),
        containerTemplate(name: 'maven', image: 'maven:3.9.6-sapmachine-17',  ttyEnabled: true, privileged: true, command: 'cat'),
    ]
    // volumes: [
    //     persistentVolumeClaim(mountPath: '/root/.npm', claimName: 'npm-pv-claim', readOnly: false),
    //     secretVolume(mountPath: '/kaniko/.docker', secretName: 'kanikodocker', readOnly: false)
    // ]
    ) {

    node(POD_LABEL) {

        try {
            stage('Checkout') {
                checkout scm
                }
            stage("SonarQube Analysis") {
                CURRENT_STAGE = "${env.STAGE_NAME}"
                dir("service") {
                    container(name: 'sonar-scanner') {
                            sh """
                            echo \${ref}
                            export branch=\$(echo \${ref} | cut -d '/' -f 3)
                            echo \${branch}
                            sonar-scanner -Dsonar.projectKey=demo-react -Dsonar.pullrequest.branch="add/something" -Dsonar.pullrequest.provider=GitHub -Dsonar.pullrequest.github.repository=quangno129/docker-reactjs-demo -Dsonar.pullrequest.key=${env.CHANGE_ID}  -Dsonar.host.url=https://sonar-demo.waterbridgepoc.com -Dsonar.login=sqa_f0839d99e6093851d7f37888385a7f10d52c20cf
                            """
                        }
                    }
                }
                stage("send report sonar") {

                container(name: 'maven') {
                    withSonarQubeEnv('SonarCloud') {
                    sh """
                    mvn sonar:sonar -Dsonar.host.url=https://sonar-demo.waterbridgepoc.com -Dsonar.scm.revision=${env.SHA} -Dsonar.login=sqa_f0839d99e6093851d7f37888385a7f10d52c20cf  -Dsonar.pullrequest.provider=GitHub  -Dsonar.pullrequest.github.repository=${env.SERVICE_REPO_URL}  -Dsonar.pullrequest.key=${env.CHANGE_ID}   -Dsonar.pullrequest.branch="add/something" -Dsonar.projectKey=demo-react
                    """
                    }
                    }
                }
            // }
        } catch (e) {
            currentBuild.result = 'FAILURE'
        }
    }
}
