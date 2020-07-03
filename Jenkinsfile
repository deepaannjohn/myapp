node('docker_ce') {
    def repoName = 'ucv-ext-test'
    def componentName = 'ucv-ext-test'
    def majorVersion = '1.0'
    def publishBranchName = 'master'
    def buildTarget = 'npm'
    def isPlugin = true
          

    /* git */
    def gitBranch = env.BRANCH_NAME
    echo "building from branch ${gitBranch}"
    echo "build target is '${buildTarget}'"
    def isPublishBranch = gitBranch == publishBranchName || gitBranch == null
    def gitResult
    def gitUrl
    def gitCommit
    def gitPR
    def author
    def prCommit

    /* docker image info */
    def tag = "${majorVersion}.${env.BUILD_ID}"
    def uniqueTag = "${gitBranch.replaceAll('-', '.')}.${tag}"
    

    /* update build info */
    currentBuild.displayName = tag
    currentBuild.description = "Built from branch ${gitBranch}"

    def packageJson
    def exceptionThrown = false
    ansiColor('xterm') {
        try {
            gitResult = checkout scm
            gitUrl = gitResult.GIT_URL
            gitCommit = gitResult.GIT_COMMIT
            gitPR = env.CHANGE_ID
            author = sh(returnStdout: true, script: "git log -1 --pretty=format:'%an'").trim()
            prCommit = sh(returnStdout: true, script: 'git rev-parse @~').trim()

            sendBuildResult('in_progress', isPublishBranch, componentName, tag, gitCommit, author)

               

            sh "echo ${tag} > service.version"
            packageJson = readJSON file: 'package.json'

            
                if (author == 'Jenkins Build' && buildTarget != 'npm-and-docker') {
                    // this was triggered from a previous jenkins build which just bumped the version #. Skip it.
                    currentBuild.result = 'SUCCESS'
                    return
                }
                sh 'node --version'
                stage('Install') {
                    sh 'npm ci --no-audit'
                }

                stage('Build') {
                    sh 'npm run build'
                                  if (isPublishBranch) {
                        archiveArtifacts artifacts: 'public/messages/*.json', allowEmptyArchive: true
                    }
                }
                
    

                
                if (isPublishBranch) {
         
                    if (buildTarget.contains('npm')) {
                        stage('Publish') {
                            
                            
                            
                                sh 'git config --global user.email "deepa.ann.john1@gmail.com"'
                                sh 'git config --global user.name "Deepa Ann John"'
                                sh 'npm version patch'
                                sh "git remote set-url origin deepaannjohn/${repoName}"
                                sh "git push origin HEAD:${gitBranch}"
                            
                            sh 'npm publish'
                        }
                    }
                }

            
       
            sendBuildResult('success', isPublishBranch, componentName, tag, gitCommit, author)
        } catch (err) {
            exceptionThrown = true
            
            println "Exception was caught in try block of jenkins job."
            sendBuildResult('failure',isPublishBranch, componentName, tag, gitCommit, author)
            println err
        } finally {
            cleanWs()
            dir("${WORKSPACE}@tmp") {
                deleteDir()
            }
            dir("${WORKSPACE}@script") {
                deleteDir()
            }
            dir("${WORKSPACE}@script@tmp") {
                deleteDir()
            }
            if (exceptionThrown) {
                error('Exception was thrown earlier')
            }
        }
    }

}


