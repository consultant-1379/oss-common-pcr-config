#!/usr/bin/env groovy
// def gerritReviewCommand = "ssh -p 29418 gerrit.ericsson.se gerrit review \${GIT_COMMIT}"
def verifications = [
        'Helm-Lint'     : 0,
        'Design-Rules'  : 0,
        'Verified'      : -1,
]
def CHART_DIR = ""
def CHART_NAME = ""
def CHART_VERSION = ""
def ADP_INCA_IMAGE = 'armdocker.rnd.ericsson.se/proj-adp-cicd-drop/adp-int-helm-chart-auto:latest'

pipeline {
    agent { label env.AGENT_LABEL }
    parameters {
        string(name: 'ARMDOCKER_USER_SECRET',
                defaultValue: 'cloudman-docker-auth-config',
                description: 'ARM Docker secret')
    }
    environment {
        SKIP_CHART_SCHEMA_VALIDATION = false
        HELM_REPO_CREDENTIALS = "${env.WORKSPACE}/repositories.yaml"
        HELM_REPO_CREDENTIALS_ID = "eoadm100_helm_repository_creds"
        KUBEVAL_KINDS_TO_SKIP = "HTTPProxy,ServerCertificate,InternalCertificate,ClientCertificate,CertificateAuthority,adapter,attributemanifest,AuthorizationPolicy,CustomResourceDefinition,DestinationRule,EnvoyFilter,Gateway,handler,HTTPAPISpec,HTTPAPISpecBinding,instance,PeerAuthentication,QuotaSpec,QuotaSpecBinding,RbacConfig,RequestAuthentication,rule,ServiceEntry,ServiceRole,ServiceRoleBinding,Sidecar,Telemetry,template,VirtualService,WorkloadEntry,WorkloadGroup"
    }

    //#step 1 :package chart
    //#step 2: K8s checks
    //#step 3: helm testsuite (reusable image)
    //#step 4: design rules

    stages {
        stage('Clean Workspace') {
            steps {
              sh "git clean -xdff"
            }
        }

        stage('Package Helm Charts') {
            steps {
                script {
                    withCredentials([file(credentialsId: env.HELM_REPO_CREDENTIALS_ID, variable: 'HELM_REPO_CREDENTIALS_FILE')]) {
                        sh "install -m 600 ${HELM_REPO_CREDENTIALS_FILE} ${env.HELM_REPO_CREDENTIALS}"
                        CHART_DIR = sh (
                            script: 'find . -name Chart.yaml -printf \'%h\n\' | uniq | head -n 1',
                            returnStdout: true
                        ).trim()
                        // CHART_DIR = sh ( // dummy
                        //     script: 'pwd',
                        //     returnStdout: true
                        // ).trim()
                        if (CHART_DIR == '') {
                            error('No chart found in this repo')
                        }
                        // sh """
                        // echo test var $CHART_DIR
                        // """
                        CHART_NAME = sh (
                            script:"""
                            tail -2 $CHART_DIR/Chart.yaml | grep "name:" | awk '{print \$2}'
                            """,
                            returnStdout: true
                        )
                        CHART_VERSION = sh (
                            script: """
                            tail -1 $CHART_DIR/Chart.yaml | grep "version:" | awk '{print \$2}'
                            """,
                            returnStdout: true
                        )
                        sh """
                        docker run -it --init \
                        --env HELM_REPO_CREDENTIALS \
                        --env CI_HELM=true \
                        $ADP_INCA_IMAGE \
                        ihc-package --version $CHART_VERSION --output . --helm-credentials ${env.HELM_REPO_CREDENTIALS} --folder $CHART_DIR
                        """
                    }
                }
            }
        }

        // stage('Kubernetes Range Compatibility Tests') {
        //     steps {
        //         script {
        //             withCredentials([file(credentialsId: params.ARMDOCKER_USER_SECRET, variable: 'DOCKERCONFIG')]) {
        //                 sh "install -m 600 ${DOCKERCONFIG} ${HOME}/.docker/config.json"
        //                 sh "${bob} run-kubernetes-compatibility-tests:chmod-execute-support-kubernetes-versions"
        //                 sh "${bob} run-kubernetes-compatibility-tests:chmod-execute-generate-helm-templates"
        //                 sh "${bob} run-kubernetes-compatibility-tests:chmod-execute-kubeval"
        //                 sh "${bob} run-kubernetes-compatibility-tests:chmod-execute-deprek8ion"
        //                 sh "${bob} run-kubernetes-compatibility-tests"
        //             }
        //         }
        //     }
        // }

        // stage('Run Helm Chart Testsuite'){
        //     steps {
        //         script {
        //             withCredentials([file(credentialsId: params.ARMDOCKER_USER_SECRET, variable: 'DOCKERCONFIG')]) {
        //                 sh "install -m 600 ${DOCKERCONFIG} ${HOME}/.docker/config.json"
        //                 sh "${bob} run-chart-testsuite"
        //             }
        //         }
        //     }
        //     post {
        //         always {
        //             sh "${bob} test-suite-report-and-clean"
        //             archiveArtifacts artifacts: 'chart-test-report.html', allowEmptyArchive: true
        //         }
        //     }
        // }

        // stage('Design Rules Check') {
        //     steps {
        //         withCredentials([file(credentialsId: 'ARTIFACTORY_TOKENS', variable: 'HELM_REPO_CREDENTIALS_FILE')]) {
        //             sh "install -m 600 ${HELM_REPO_CREDENTIALS_FILE} ${HELM_REPO_CREDENTIALS}"
        //             sh "${bob} set-design-rule-parameters design-rule-checker"
        //         }
        //     }
        //     post {
        //         always {
        //             archiveArtifacts 'design-rule-check-report.html'
        //         }
        //         success {
        //             script {
        //                 verifications['Design-Rules'] = +1
        //             }
        //         }
        //         failure {
        //             script {
        //                 verifications['Design-Rules'] = -1
        //             }
        //         }
        //     }
        // }
    }
    // post { //TODO: uncomment this when job will be ready to do auto review
    //     success {
    //         script {
    //             verifications['Verified'] = +1
    //             def labelArgs = verifications
    //                     .collect { entry -> "--label ${entry.key}=${entry.value}" }
    //                     .join(' ')
    //             sh "${gerritReviewCommand} ${labelArgs}"
    //         }
    //     }
    //     failure {
    //         script {
    //             def labelArgs = verifications
    //                     .collect { entry -> "--label ${entry.key}=${entry.value}" }
    //                     .join(' ')
    //             sh "${gerritReviewCommand} ${labelArgs}"
    //         }
    //     }
    // }
}
