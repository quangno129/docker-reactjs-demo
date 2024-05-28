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
        containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-8',  ttyEnabled: true, privileged: true, command: 'cat'),
    ]
    // volumes: [
    //     persistentVolumeClaim(mountPath: '/root/.npm', claimName: 'npm-pv-claim', readOnly: false),
    //     secretVolume(mountPath: '/kaniko/.docker', secretName: 'kanikodocker', readOnly: false)
    // ]
    ) {

    node(POD_LABEL) {

        try {
            // stage ('Example') {
            //     sh """
            //         git clone -b $branch https://${env.PERSONAL_ACCESS_TOKEN}@${env.SERVICE_REPO_URL} service --depth 1
            //     """
            //     }
            stage('Checkout') {
                // CURRENT_STAGE = "${env.STAGE_NAME}"
                // withCredentials([string(credentialsId: 'github-token', variable: 'PERSONAL_ACCESS_TOKEN')]) {
                //     sh """
                //         git clone https://${env.PERSONAL_ACCESS_TOKEN}@${env.SERVICE_REPO_URL} service --depth 1
                //     """
                // }
                    checkout scm

            }

            // stage("Extract version") {
            //     CURRENT_STAGE = "${env.STAGE_NAME}"
            //     VERSION = "${VERSION}.${env.BUILD_NUMBER}"
            //     currentBuild.displayName = "${VERSION}"
            //     if (params.BRANCH_TO_BUILD != 'main') {
            //         currentBuild.description = "From the branch: ${params.BRANCH_TO_BUILD}"
            //     }
            //     addShortText(text: "${params.BRANCH_TO_BUILD}", border: 0, background: 'yellow')

            //     sh """
            //         # update version to package.json
            //         sed -i -e 's/"version": .*"/"version": '\\\"${VERSION}\\\"'/g' service/package.json
            //     """
            // }

            stage("SonarQube Analysis") {
                CURRENT_STAGE = "${env.STAGE_NAME}"
                dir("service") {
                    container(name: 'sonar-scanner') {
                            sh """
                            echo \${ref}
                            export branch=\$(echo \${ref} | cut -d '/' -f 3)
                            echo \${branch}
                            sonar-scanner -Dsonar.projectKey=demo-react -Dsonar.pullrequest.branch=\${branch} -Dsonar.pullrequest.provider=GitHub -Dsonar.pullrequest.github.repository=quangno129/docker-reactjs-demo -Dsonar.pullrequest.key=5 -Dsonar.host.url=https://sonar-demo.waterbridgepoc.com -Dsonar.login=sqa_f0839d99e6093851d7f37888385a7f10d52c20cf
                            """
                        }
                    }
                }
                stage("send report sonar") {

                container(name: 'maven') {
                    withSonarQubeEnv('SonarCloud') {
                    sh """mvn sonar:sonar -Dsonar.pullrequest.provider=GitHub  -Dsonar.pullrequest.github.repository=${env.SERVICE_REPO_URL}  -Dsonar.pullrequest.key=${env.CHANGE_ID}   -Dsonar.pullrequest.branch=${env.BRANCH_NAME}"""
                    }
                    }
                }


            //stage("SonarQube Quality Gate") {
            //    timeout(time: 2, unit: 'MINUTES') {
            //        def qg = waitForQualityGate()
            //        if (qg.status != 'OK') {
            //            error "Pipeline aborted due to quality gate failure: ${qg.status}"
            //        }
            //    }
            //}

            // stage("Build and Push Docker Image to ECR") {
            //     CURRENT_STAGE = "${env.STAGE_NAME}"
            //     dir("service") {
            //         container(name: 'kaniko', shell: '/busybox/sh') {
            //             withCredentials([string(credentialsId: 'npm-registry-token', variable: 'NPM_REGISTRY_TOKEN')]) {
            //                 sh """#!/busybox/sh
            //                     if [ "${NEED_TO_OVERRIDE_CONFIG}" = "true" ]
            //                     then
            //                         mv ../helmvalues/core/dev/${SERVICE_NAME}/${SERVICE_NAME}-${FILE_CONFIG} ${PATH_TO_CONFIG}${FILE_CONFIG}
            //                     fi

            //                     if [ "${NEED_ENV_FILE}" = "true" ]
            //                     then
            //                         mv ../helmvalues/core/dev/${SERVICE_NAME}/.env .env
            //                     fi

            //                     /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --build-arg NPM_REGISTRY_TOKEN="${NPM_REGISTRY_TOKEN}" --insecure --skip-tls-verify --cache=true --destination=${ECR_URL}/${PROJECT_NAME}/image/${SERVICE_NAME}:${VERSION}
            //                 """
            //             }
            //         }
            //     }
            // }

            // stage("Tag the version") {
            //     CURRENT_STAGE = "${env.STAGE_NAME}"
            //     dir("service") {
            //         sh """
            //             git config user.email jenkins
            //             git config user.name jenkins

            //             git add package.json
            //             git commit --no-verify -m "chore(release): v${VERSION}"
            //             git push origin "${params.BRANCH_TO_BUILD}"

            //             git tag -a "v${VERSION}" -m "From branch: ${params.BRANCH_TO_BUILD}"
            //             git push origin "v${VERSION}"
            //         """
            //     }

            //     currentBuild.result = 'SUCCESS'
            // }
        } catch (e) {
            currentBuild.result = 'FAILURE'
        }
    }
}
