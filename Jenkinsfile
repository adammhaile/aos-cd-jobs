
node {
    checkout scm
    def commonlib = load("pipeline-scripts/commonlib.groovy")

    // Expose properties for a parameterized build
    properties(
        [
            buildDiscarder(
                logRotator(
                    artifactDaysToKeepStr: '',
                    artifactNumToKeepStr: '',
                    daysToKeepStr: '',
                    numToKeepStr: '5')),
            [
                $class: 'ParametersDefinitionProperty',
                parameterDefinitions: [
                    commonlib.suppressEmailParam(),
                    [
                        name: 'MAIL_LIST_SUCCESS',
                        description: 'Success Mailing List',
                        $class: 'hudson.model.StringParameterDefinition',
                        defaultValue: [
                            'ahaile@redhat.com',
                        ].join(',')
                    ],
                    [
                        name: 'MAIL_LIST_FAILURE',
                        description: 'Failure Mailing List',
                        $class: 'hudson.model.StringParameterDefinition',
                        defaultValue: [
                            'ahaile@redhat.com',
                        ].join(',')
                    ],
                    commonlib.mockParam(),
                ]
            ],
        ]
    )

    // currentBuild.displayName = "#${currentBuild.number} - ${version}-${release}"

    try {
        sshagent(["openshift-bot"]) {
            // To work on real repos, buildlib operations must run with the permissions of openshift-bot
            currentBuild.description = ""

            stage("Get latest release") {
                withCredentials([string(credentialsId: 'AHAILE_GITHUB_AUTH_TOKEN', variable: 'TOKEN')]) {
                    release_url_cmd = "curl -s https://api.github.com/repos/openshift/doozer/releases/latest?access_token=${TOKEN} | jq '.zipball_url'"
                    release_url = sh(returnStdout: true, script: release_url_cmd).trim()
                    echo release_url
                }
            }

            // commonlib.email(
            //     to: "${params.MAIL_LIST_SUCCESS}",
            //     from: "aos-team-art@redhat.com",
            //     subject: "Successful custom OCP build: ${currentBuild.displayName}",
            //     body: "Jenkins job: ${env.BUILD_URL}\n${currentBuild.description}",
            // )
        }
    } catch (err) {
        currentBuild.description += "\nerror: ${err.getMessage()}"
//         commonlib.email(
//             to: "${params.MAIL_LIST_FAILURE}",
//             from: "aos-team-art@redhat.com",
//             subject: "Error building custom OCP: ${currentBuild.displayName}",
//             body: """Encountered an error while running OCP pipeline:

// ${currentBuild.description}

// Jenkins job: ${env.BUILD_URL}
// Job console: ${env.BUILD_URL}/console
//     """)

        currentBuild.result = "FAILURE"
        throw err
    } finally {
    }
}
