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

                stage('Checkout elite-ui Repo') {
                    git branch: 'main', url: 'https://github.com/NIUANULP/nulp-elite-ui', changelog: false, poll: false
                }

                stage('Build') {
                    // Define the Node.js version to use
                    def NODE_VERSION = '16' // Adjust this to your desired Node.js version
                    def NVM_DIR = '/var/lib/jenkins/.nvm'

                    // Install dependencies and build
                    sh """
                        #!/bin/bash
                        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        export NVM_DIR="$HOME/.nvm"
                        if [ -s "$NVM_DIR/nvm.sh" ]; then
                            . "$NVM_DIR/nvm.sh"
                        fi
                        if [ -s "$NVM_DIR/bash_completion" ]; then
                            . "$NVM_DIR/bash_completion"
                        fi
                        nvm install $NODE_VERSION
                        nvm use $NODE_VERSION
                        yarn install
                        yarn build
                    """
                }

                stage('Copy Artifacts from elite-ui Repo to angular Repo') {
                    sh """
                    cp -r nulp-elite-ui/prod-build/* /var/lib/jenkins/workspace/Build/Core/Player/src/app/app_dist/dist/
                    """
                }

                stage('Build angular Repo') {
                    sh("bash ./build.sh ${build_tag} ${env.NODE_NAME} ${hub_org} ${params.buildDockerImage} ${params.buildCdnAssests} ${params.cdnUrl}")
                }

                stage('Archive Artifacts') {
                    archiveArtifacts artifacts: "metadata.json"
                    if (params.buildCdnAssests == 'true') {
                        sh """
                        rm -rf cdn_assets
                        mkdir cdn_assets
                        cp -r src/app/dist-cdn/* cdn_assets/
                        zip -Jr cdn_assets.zip cdn_assets
                        """
                        archiveArtifacts artifacts: "src/app/dist-cdn/index_cdn.ejs, cdn_assets.zip"
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
