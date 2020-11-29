pipeline {
    agent any
    parameters {
        choice(name: 'branch', choices: ['main', 'staging', 'dev'], description: '')
        string(name: 'accesskey', defaultValue: '', description: 'Access Key for aws')
        password(name: 'secretkey', defaultValue: 'SECRET', description: 'Secret key for aws')
        string(name: 's3bucket', defaultValue: '', description: 'S3 bucket for artifact')
        string(name: 'buildproject', defaultValue: '', description: 'Code Build Project name')
        string(name: 'artifact', defaultValue: '', description: 'Artifact Name with extension')
    }
    triggers {
        cron('H */4 * * *')
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
                echo "Prebuild stage ${params.s3bucket}"
                
            }
        }
        stage("build"){
            steps{
                echo "Build Stage ${params.buildproject}"
                
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