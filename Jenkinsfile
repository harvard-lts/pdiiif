#!groovy
@Library('lts-basic-pipeline@forked-repo-pipeline') _

// projName: The directory name for the project on the servers for it's docker/config files
// intTestPort: Port of integration test container
// intTestEndpoints: List of integration test endpoints i.e. ['healthcheck/', 'another/example/']
// default values: slackChannel = "lts-jenkins-notifications"

def endpoints = []
ltsBasicPipeline.call("mps-pdiiif", "MPS-VIEWER", "mps/mps-viewer", "", endpoints, "lts-mps-viewer-alerts") 
