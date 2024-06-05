node('') {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

        ansiColor('xterm') {
            timestamps {
                stage('Checkout First Repo') {
                    if (!env.hub_org) {
                        println(ANSI_BOLD + ANSI_RED + "Uh Oh! Please set a Jenkins environment variable named hub_org with value as registery/sunbidrded" + ANSI_NORMAL)
                        error 'Please resolve the errors and rerun..'
                    } else {
                        println(ANSI_BOLD + ANSI_GREEN + "Found environment variable named hub_org with value as: " + hub_org + ANSI_NORMAL)
                    }
                    // cleanWs()
                    checkout scm
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    build_tag = sh(script: "echo " + params.github_release_tag.split('/')[-1] + "_" + commit_hash + "_" + env.BUILD_NUMBER, returnStdout: true).trim()
                    echo "build_tag: " + build_tag
                }

                stage('Checkout Second Repo') {
                    git url: 'https://github.com/NIUANULP/nulp-elite-ui', branch: 'main', changelog: false, poll: false
                }

                stage('Build Second Repo') {
                    dir('second-repo') {
                        sh "chmod +x ./scripts/pack-prod-build.sh"
                        sh "bash ./scripts/pack-prod-build.sh"
                    }
                }

                stage('Copy Artifacts from Second Repo to First Repo') {
                    sh """
                    cp -r nulp-elite-ui/prod-build/webapp/* /var/lib/jenkins/workspace/Build/Core/Player/src/app/app_dist/dist/
                    """
                }

                stage('Build First Repo') {
                    sh("bash ./build.sh ${build_tag} ${env.NODE_NAME} ${hub_org} ${params.buildDockerImage} ${params.buildCdnAssests} ${params.cdnUrl}")
                }

                stage('Archive Artifacts') {
                    archiveArtifacts "metadata.json"
                    if (params.buildCdnAssests == 'true') {
                        sh """
                        rm -rf cdn_assets
                        mkdir cdn_assets
                        cp -r src/app/dist-cdn/* cdn_assets/
                        zip -Jr cdn_assets.zip cdn_assets
                        """
                        archiveArtifacts "src/app/dist-cdn/index_cdn.ejs, cdn_assets.zip"
                    }
                    currentBuild.description = "${build_tag}"
                }
            }
        }
    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }
}
