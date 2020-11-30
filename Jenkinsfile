pipeline {
    agent any
    environment{
        artifact = "testapp_build.zip"
    }
    parameters {
        choice(name: "branch", choices: ["main", "staging", "dev"], description: "")
        string(name: "s3bucket", defaultValue: "", description: "S3 bucket for artifact")
        string(name: "stackname", defaultValue: "", description: "Cloudformation stack name")
        string(name: "buildproject", defaultValue: "", description: "Code Build Project name")
        string(name: "region", defaultValue: "eu-central-1", description: "Artifact Name with extension")
        string(name: "codedeployapp", defaultValue: "", description: "Artifact Name with extension")
        string(name: "codedeploygroup", defaultValue: "", description: "Artifact Name with extension")
    }
    triggers {
        cron("H */4 * * *")
    }
    stages{
        stage("setenvironment"){
            steps{
                switch(env.branch) {
                    case 'staging':
                        env.awscredentialId = "staging-credentials-id"
                        break
                    case 'dev':
                        env.awscredentialId = "dev-credentials-id"
                        break 
                    case 'main':
                        env.awscredentialId = "d85fd0d8-4b1f-4ddd-a32d-66bd62c3edda"
                        break        
                }
            }
        }
        stage("getstack"){
            steps{
                withAWS(credentials: "${awscredentialId}",region: "${region}"){
                    def outputs = cfnDescribe(stack:"${stackname}")
                    echo "${output}"
                }
            }
        }
        stage("checkout"){
            steps{
                echo "checkout the git repo from branch ${params.branch}"
                git url: "https://github.com/radhika-pr/sample-application.git" , branch: "${params.branch}"
            }
        }
        stage("prebuild"){
            steps{
                echo "AWS CodeBuild Config to follow"
                withAWS(credentials: "${awscredentialId}",region: "${region}"){
                    sh 'echo "CodeBuild Block"'
                    awsCodeBuild artifactEncryptionDisabledOverride: "", artifactLocationOverride: "", artifactNameOverride: "", artifactNamespaceOverride: "", artifactPackagingOverride: "", artifactPathOverride: "", artifactTypeOverride: "", buildSpecFile: "", buildTimeoutOverride: "", cacheLocationOverride: "", cacheModesOverride: "", cacheTypeOverride: "", certificateOverride: "", cloudWatchLogsGroupNameOverride: "", cloudWatchLogsStatusOverride: "", cloudWatchLogsStreamNameOverride: "", computeTypeOverride: "", credentialsId: "", credentialsType: "keys", cwlStreamingDisabled: "", downloadArtifacts: "false", downloadArtifactsRelativePath: "", envParameters: "", envVariables: "", environmentTypeOverride: "", exceptionFailureMode: "", gitCloneDepthOverride: "", imageOverride: "", insecureSslOverride: "", localSourcePath: "", overrideArtifactName: "", privilegedModeOverride: "", projectName: "${params.buildproject}", proxyHost: "", proxyPort: "", region: "${params.region}", reportBuildStatusOverride: "", s3LogsEncryptionDisabledOverride: "", s3LogsLocationOverride: "", s3LogsStatusOverride: "", secondaryArtifactsOverride: "", secondarySourcesOverride: "", secondarySourcesVersionOverride: "", serviceRoleOverride: "", sourceControlType: "jenkins", sourceLocationOverride: "", sourceTypeOverride: "", sourceVersion: "", sseAlgorithm: "", workspaceSubdir: ""
                    sh 'echo "download zip file"'
                    s3Download(bucket: "${params.s3bucket}", file: "${artifact}", path: "${artifact}",force:true)
                }    
                echo "Clean everything copied from git repo"
                fileOperations([fileDeleteOperation(
                    excludes: "${artifact}",
                    includes: "*"
                )])
            }
        }
        stage("build"){
            when {
            branch 'main'
            }
            steps{
                withAWS(credentials: "${awscredentialId}", region: "${region}"){
                    createDeployment(
                        s3Bucket: "${params.s3bucket}",
                        s3Key: "${artifact}",
                        s3BundleType: "zip", // [Valid values: tar | tgz | zip | YAML | JSON]
                        applicationName: "${codedeployapp}",
                        deploymentGroupName: "${codedeploygroup}",
                        deploymentConfigName: "CodeDeployDefault.OneAtATime",
                        description: "Test deploy",
                        waitForCompletion: "true",
                        ignoreApplicationStopFailures: "false",
                        fileExistsBehavior: "OVERWRITE"
                     )

                }
            }
        }
        stage("postbuild"){
            steps{
                echo "Clean Workspace"
                cleanWs()
            }
        }
    }
}