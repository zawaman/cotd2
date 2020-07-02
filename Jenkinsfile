// Set project names test
def testProj   = "cicd-test"
def prodProj   = "cicd-prod"
def svc_name   = "cotd2"
pipeline {
  agent any
  stages{
    stage("Build"){
      steps{
        echo '*** Build Starting ***'
        script{
          openshift.withCluster() {
            openshift.withProject("${testProj}") {
              openshift.selector('bc', 'cotd2').startBuild("--wait").logs('-f')
            }
          }
        echo '*** Build Complete ***'
        }
      }
    }
    stage ('Deploy in Testing Env') {
      steps{
        echo '*** Deployment Starting ***'
        script{
          openshift.withCluster() {
            openshift.withProject("${testProj}") {
              // Deploy the cotd application in the testProject
              openshift.selector('dc', 'cotd2').rollout().latest();
              // Wait for application to be deployed
              def dc = openshift.selector("dc", "cotd2").object()
              while (dc.spec.replicas != dc.status.availableReplicas) {
                sleep 1
              }
              sleep 30
            }
          }
        }
        echo '*** Deployment Complete ***'
      }
    }
    stage ('Test in Testing Env') {
      steps{
        echo '*** Service Verification Starting ***'
        script{
          openshift.withCluster() {
            openshift.withProject("${testProj}") {
              def connected = openshift.verifyService("${svc_name}")
              if (connected) {
                echo "Able to connect to ${svc_name}"
              } else {
                echo "Unable to connect to ${svc_name}"
              }
              // curl the testProject route to get cats
              def route = openshift.selector("route", "cotd2").object()
              def the_route = "${route.spec.host}"
              echo "route: ${the_route}"
              sh "curl -s http://${the_route}/item.php | grep cities"
            }
          }
        }
        echo '*** Service Verification Complete ***'
      }
    }
    stage ('Promote and Verify in Production Env') {
      steps{
        echo '*** Waiting for Input ***'
        script{
          openshift.withCluster() {
            openshift.withProject("${prodProj}") {
              input message: 'Should we deploy to Production?', ok: "Promote"
              echo '*** Deploying to Production ***'
              openshift.tag("${testProj}/cotd2:latest", "${testProj}/cotd2:prodready")
              sleep 10
              openshift.selector('dc', 'cotd2').rollout().latest();
              def dc = openshift.selector("dc", "cotd2").object()
              while (dc.spec.replicas != dc.status.availableReplicas) {
                sleep 1
              }
              sleep 30
              // test route
              def route = openshift.selector("route", "cotd2").object()
              def the_route = "${route.spec.host}"
              echo "route: ${the_route}"
              sh "curl -s http://${the_route}/item.php | grep cities"
            }
          }
        }
      }
    }
  }
}
