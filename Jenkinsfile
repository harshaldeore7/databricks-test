// Filename: Jenkinsfile
node {
  def GITREPOREMOTE = "https://github.com/harshaldeore7/databricks-test.git"
  def GITBRANCH     = "main"
  def DBCLIPATH     = "/usr/local/bin"
  def JQPATH        = "/usr/bin"
  def JOBPREFIX     = "testing_job"
  def BUNDLETARGET  = "dev"

  stage('Checkout') {
    git branch: GITBRANCH, url: GITREPOREMOTE
  }
  stage('Get Working Directory') {
  script {
    env.WORKING_DIR = sh(script: 'pwd', returnStdout: true).trim()  // Capture current working directory
    echo "Working Directory is: ${env.WORKING_DIR}"
   }
  }
  stage('Validate Bundle') {
    sh """#!/bin/bash
          ${DBCLIPATH}/databricks bundle validate -t ${BUNDLETARGET}
       """
  }
  stage('Deploy Bundle') {
    sh """#!/bin/bash
          ${DBCLIPATH}/databricks bundle deploy -t ${BUNDLETARGET}
       """
  }
  stage('Run Unit Tests') {
    sh """#!/bin/bash
          ${DBCLIPATH}/databricks bundle run -t ${BUNDLETARGET} run-unit-tests
       """
  }
  stage('Run Notebook') {
    sh """#!/bin/bash
          ${DBCLIPATH}/databricks bundle run -t ${BUNDLETARGET} run-dabdemo-notebook
       """
  }
  stage('Evaluate Notebook Runs') {
    sh """#!/bin/bash
          ${DBCLIPATH}/databricks bundle run -t ${BUNDLETARGET} evaluate-notebook-runs
       """
  }
  stage('Import Test Results') {
    def DATABRICKS_BUNDLE_WORKSPACE_ROOT_PATH
    def getPath = "${DBCLIPATH}/databricks bundle validate -t ${BUNDLETARGET} | ${JQPATH}/jq -r .workspace.file_path"
    def output = sh(script: getPath, returnStdout: true).trim()

    if (output) {
      DATABRICKS_BUNDLE_WORKSPACE_ROOT_PATH = "${output}"
    } else {
      error "Failed to capture output or command execution failed: ${getPath}"
    }

    sh """#!/bin/bash
          ${DBCLIPATH}/databricks workspace export-dir \
          ${DATABRICKS_BUNDLE_WORKSPACE_ROOT_PATH}/Validation/Output/test-results \
          ${WORKSPACE}/Validation/Output/test-results \
          -t ${BUNDLETARGET} \
          --overwrite
       """
  }
  stage('Publish Test Results') {
    junit allowEmptyResults: true, testResults: '**/test-results/*.xml', skipPublishingChecks: true
  }
}
