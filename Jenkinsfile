import groovy.json.JsonSlurper

pipeline{
    agent any

    environment {
        timestamp = new java.util.Date().format("yyyyMMdd-HHmm", TimeZone.getTimeZone('Asia/Shanghai'));
        Server_URL = "https://www.jenkins.io";
        Jenkins_Auth = "Jenkins_Credential_ID";
    }

    parameters {
        string(name: "nodeLabel", defaultValue: "windows_node", description: "The target build node label for node selection")
        string(name: "targetJobs", defaultValue: "", description: "The target jobs to schedule, use ',' to separate multiple jobs, each job should exists on the jenkins server")
        string(name: "executorsRemain", defaultValue: "1", description: "Keep idle executors for other jobs")
    }

    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: "7", artifactNumToKeepStr: "7", daysToKeepStr: "7", numToKeepStr: "7"))
        disableConcurrentBuilds()
        skipDefaultCheckout()
        timestamps()
        timeout(time: 10, unit: 'HOURS')
    }

    stages {
        stage("Scheduling Jobs"){
            steps {
                script {
                    echo "Start to schedule jobs when executors available"
                    def target_jobs = params.targetJobs.tokenize(",");
                    def executors_threshold = 1;
                    if(params.executorsRemain.isInteger()){
                        executors_threshold = params.executorsRemain as Integer;
                    }

                    target_jobs.each{
                        def idleExecutors = getAvailableExecutor(params.nodeLabel, Server_URL, Jenkins_Auth);
                        echo "current idel executors: ${idleExecutors}"
                        while (idleExecutors <= executors_threshold){ // we leave one executor buffer for other jobs
                            echo "wait 5 minutes for next executor..."
                            sleep(time:5,unit:"MINUTES")
                            idleExecutors = getAvailableExecutor(params.nodeLabel);
                        }

                        def job_name = it.trim();
                        build job: job_name, wait: false
                        echo "scheduled job: ${job_name}"
                        sleep(time:10,unit:"SECONDS")
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                try {
                    echo "build completed ${currentBuild.result}"
                }finally {
                    deleteDir()
                }
            }
        }
    }
}

def getAvailableExecutor(String nodeLable,String Server_URL,String Jenkins_Auth){
    def rx = httpRequest authentication: "${Jenkins_Auth}", url: "${Server_URL}/label/${nodeLable}/api/json"
    def rxJson = new JsonSlurper().parseText(rx.getContent())
    return rxJson['idleExecutors']
}