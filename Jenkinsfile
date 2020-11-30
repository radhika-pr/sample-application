def outputs
pipeline {
    agent any
    environment{
        artifact = "testapp_build.zip"
        buildproject = "sampleproject"
        codedeployapp = "deployapp"
        codedeploygroup = "samplegroup"
        s3bucket ="cfns3"
        dns = "elb-dns"
    }
    parameters {
        choice(name: "branch", choices: ["main", "staging", "dev"], description: "")
        string(name: "stackname", defaultValue: "", description: "Cloudformation stack name")
        string(name: "region", defaultValue: "eu-central-1", description: "Artifact Name with extension")
    }
    triggers {
        cron("H */4 * * *")
    }
    stages{
        stage("set Authentication"){
            steps{
                script {
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
        }
        stage("get CFStack"){
            steps{
                withAWS(credentials: "${awscredentialId}",region: "${region}"){
                    script {
                        outputs = cfnDescribe(stack:"${stackname}")
                    }
                }
            }
        }
        stage("checkoutSCM"){
            steps{
                script{
                    buildproject = outputs.CodeBuildProjectName
                    codedeployapp = outputs.CodeDeployApplicationName
                    codedeploygroup = outputs.CodeDeployDeploymentGroup
                    s3bucket = outputs.S3BucketName
                    dns = outputs.ELBDNSName
                }
                
                echo "checkout the git repo from branch ${params.branch}"
                git url: "https://github.com/radhika-pr/sample-application.git" , branch: "${params.branch}"
            }
        }
        stage("app Build Test"){
            steps{
                echo "AWS CodeBuild Config to follow"
                withAWS(credentials: "${awscredentialId}",region: "${region}"){
                    sh 'echo "CodeBuild Block"'
                    awsCodeBuild  credentialsType: "keys", downloadArtifacts: "false",  projectName: "${buildproject}", region: "${params.region}", sourceControlType: "jenkins"
                    sh 'echo "download zip file"'
                    s3Download(bucket: "${s3bucket}", file: "${artifact}", path: "${artifact}",force:true)
                }    
            }
        }
        stage("deploy ready"){
            steps{
                echo "Clean everything copied from git repo"
                fileOperations([fileDeleteOperation(
                    excludes: "${artifact}",
                    includes: "*"
                )])
                echo "Unzip artifact"
                fileOperations([fileUnZipOperation(
                    filePath: "${artifact}",
                    targetLocation: "./"
                )])
                echo "remove zip artifact"
                fileOperations([fileDeleteOperation(
                    includes: "${artifact}"
                )])
            }
        }
        stage("app Deployment"){
            steps{
                withAWS(credentials: "${awscredentialId}", region: "${region}"){
                    createDeployment(
                        s3Bucket: "${s3bucket}",
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
        stage("post Deployment"){
            steps{
                script {
                    echo "sleep for 5min to get application ready"
                    sleep 300
                    final String url = "http://${dns}"
                    def response = httpRequest "${url}"

                    if (response != 200 && status != 201) {
                         error("Returned status code = $status when calling $url")
                    }
                    echo response
                }
                echo "Clean Workspace"
                cleanWs()
            }
        }
    }
}