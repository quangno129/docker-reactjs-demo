podTemplate(
    envVars: [
        envVar(key: 'PROJECT_NAME', value: "cxp"),
        envVar(key: 'SERVICE_NAME', value: "docker-reactjs-demo"),
        envVar(key: 'SERVICE_REPO_URL', value: "github.com/quangno129/docker-reactjs-demo.git"),
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
    ) {

    node(POD_LABEL) {

        try {
            stage('Checkout') {
//                 steps{
//         script {
//         def isPr() {
//         env.CHANGE_ID != null
//     }

// // github-specific refspec
//     def refspec = "+refs/pull/${env.CHANGE_ID}/head:refs/remotes/origin/PR-${env.CHANGE_ID} +refs/heads/master:refs/remotes/origin/master"

//     def extensions = []
//     if (isPr()) {
//         extensions = [[$class: 'PreBuildMerge', options: [mergeRemote: "refs/remotes/origin", mergeTarget: "PR-${env.CHANGE_ID}"]]]
//     }

            checkout([
                class: 'GitSCM',
                doGenerateSubmoduleConfigurations: false,
                extensions:  [[$class: 'PreBuildMerge', options: [mergeRemote: "refs/remotes/origin", mergeTarget: "PR-${env.CHANGE_ID}"]]],
                submoduleCfg: [],
                userRemoteConfigs: [[
                    refspec: "+refs/pull/${env.CHANGE_ID}/head:refs/remotes/origin/PR-${env.CHANGE_ID} +refs/heads/master:refs/remotes/origin/master",
                    credentialsId: 'github-credential',
                    url: ${env.SERVICE_REPO_URL}
                ]]
            ])
                }
            stage("SonarQube Analysis") {
                CURRENT_STAGE = "${env.STAGE_NAME}"
                dir("service") {
                    container(name: 'sonar-scanner') {
                            sh """
                            echo \${ref}
                            export branch=\$(echo \${ref} | cut -d '/' -f 3)
                            echo \${branch}
                            sonar-scanner -Dsonar.projectKey=demo-react -Dsonar.pullrequest.branch=${env.BRANCH_NAME} -Dsonar.pullrequest.provider=GitHub -Dsonar.pullrequest.github.repository=quangno129/docker-reactjs-demo -Dsonar.pullrequest.key=5 -Dsonar.host.url=https://sonar-demo.waterbridgepoc.com -Dsonar.login=sqa_f0839d99e6093851d7f37888385a7f10d52c20cf
                            """
                        }
                    }
                }
                stage("send report sonar") {

                container(name: 'maven') {
                    withSonarQubeEnv('SonarCloud') {
                    sh """
                    mvn sonar:sonar -Dsonar.host.url=https://sonar-demo.waterbridgepoc.com -Dsonar.login=sqa_f0839d99e6093851d7f37888385a7f10d52c20cf  -Dsonar.pullrequest.provider=GitHub  -Dsonar.pullrequest.github.repository=${env.SERVICE_REPO_URL}  -Dsonar.pullrequest.key=${env.CHANGE_ID}   -Dsonar.pullrequest.branch=${env.BRANCH_NAME} -Dsonar.projectKey=demo-react
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
