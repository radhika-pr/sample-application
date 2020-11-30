pipeline {
    agent any
    parameters {
        choice(name: "branch", choices: ["main", "staging", "dev"], description: "")
        string(name: "s3bucket", defaultValue: "", description: "S3 bucket for artifact")
        string(name: "buildproject", defaultValue: "", description: "Code Build Project name")
        string(name: "region", defaultValue: "eu-central-1", description: "Artifact Name with extension")
        string(name: "artifact", defaultValue: "", description: "Artifact Name with extension")
    }
    triggers {
        cron("H */4 * * *")
    }
    stages{
        stage("checkout"){
            steps{
                echo "checkout the git repo from branch ${params.branch}"
                git url: "https://github.com/radhika-pr/sample-application.git" , branch: "${params.branch}"
            }
        }
        stage("prebuild"){
            steps{
                echo "AWS CodeBuild Config to follow"
                withAWS(role:"jenkins-instance-role",region: "${region}"){
                    sh 'echo "CodeBuild Block"'
                    awsCodeBuild artifactEncryptionDisabledOverride: "",artifactLocationOverride: "",artifactNameOverride: "", artifactNamespaceOverride: "",artifactPackagingOverride: "", artifactPathOverride: "", artifactTypeOverride: "",awsAccessKey: "${params.accessKey}", awsSecretKey: "${params.secretKey}",credentialsId: "", credentialsType: "keys",cwlStreamingDisabled: "", downloadArtifacts: "false",projectName: "${params.buildproject}", region: "${params.region}",sourceControlType: "jenkins"
                    sh 'echo "download zip file"'
                    s3Download bucket: "${params.s3bucket}", file: "${artifact}", path: "${artifact}"
                }    
                echo "Clean everything copied from git repo"
                fileOperations([fileDeleteOperation(
                    includes: "*"
                )])
            }
        }
        stage("build"){
            steps{
                echo "HTTP request to S3 bucket"
                echo "File Unzip"
                fileOperations([fileUnZipOperation(
                    filePath: "${artifact}",
                    targetLocation: "./"
                )])
                echo "zip file delete"
                fileOperations([fileDeleteOperation(
                    includes: "${artifact}"
                )])
                withAWS(role:"jenkins-instance-role", region: "${region}"){
                    createDeployment(
                        s3Bucket: "${params.s3bucket}",
                        s3Key: "artifacts/SimpleWebApp.zip",
                        s3BundleType: "zip", // [Valid values: tar | tgz | zip | YAML | JSON]
                        applicationName: "",
                        deploymentGroupName: "SampleDeploymentGroup",
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