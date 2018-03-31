# AppDynamics Daemonset for Openshift
## Introduction
This repository contains a sample DaemonSet Deployment File to deploy an AppDynamics Machine Agent and Java Agent within RedHat OpenShift or OpenShift Origin.

The DaemonSet will monitor each **Host** within the OpenShift Cluster and the corresponding **Containers** that have an AppDynamics Application Agent running.

The Application Agent for Java will also be provided by the DaemonSet and be mounted into the corresponding Application PODs. There are also other alternatives on how to deploy Application Agent within OpenShift depending on your Security requirements.

From a license perspective you need a **Server Visibility License** to use this example.
## Supported Technologies/Agents
| Platform | Application Agent | Machine Agent | Analytics Agent |
|:--------:|:-----------------:|:-------------:|:---------------:|
| Java     | Yes               | Yes           | WIP             |
## Repository Structure
| Folder                                           | Description                                                     |
|--------------------------------------------------|-----------------------------------------------------------------|
| `_common`                                        | Files that span across each platform                            |
| `_common/.ebextensions/nginx`                    | Nginx configuration for the sample application                  |
| `<platform>-(machine)-(analytics)`               | Platform and Agent specific scripts                             |
| `<platform>-(machine)-(analytics)/.ebextensions` | Platform specific AWS Elastic Beanstalk Extension Configuration |
| `<platform>-(machine)-(analytics)/testing`       | Testing Script for Linux                                        |
## Requirements
* OpenShift Cluster up and running
* `oc` CLI installed
* `git` installed

## Deploy the DaemonSet
* Start OpenShift if needed
  * ```oc cluster up --public-hostname=<YOUR Public IP>```
* Login
  * ```oc login -u system:admin```
* Create Service Account (This is needed to do some privileged stuff in OpenShift)
  * ```oc create serviceaccount appd```
  * ```oc policy add-role-to-user view -z appd```
  * ```oc adm policy add-scc-to-user privileged -z appd```
* Configure the DaemonSet by adjusting the Environment Variables
    ```
    - env:
      - name: APPDYNAMICS_CONTROLLER_HOST_NAME
        value: ""
      - name: APPDYNAMICS_CONTROLLER_PORT
        value: ""
      - name: APPDYNAMICS_AGENT_ACCOUNT_NAME
        value: "customer1"
      - name: APPDYNAMICS_AGENT_ACCOUNT_ACCESS_KEY
        value: ""
      - name: APPDYNAMICS_SIM_ENABLED
        value: "true"
      - name: APPDYNAMICS_DOCKER_ENABLED
        value: "true"
      - name: APPDYNAMICS_CONTROLLER_SSL_ENABLED
        value: "false"
    ```
* **Optional** Change the Volume Mount from hostPath to something else
    ```
    volumes:
      - hostPath:
          path: /
        name: host-root
      - hostPath:
          path: /opt/app-agent
        name: app-agent
      - hostPath:
          path: /var/run/docker.sock
        name: docker-sock
    ```
* Deploy the Agent
  * ```oc create -f appd-agents.yml```

## Deploy a Sample App
