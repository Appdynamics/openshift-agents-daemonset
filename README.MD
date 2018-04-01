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
| `appd-agents.yml`                                | OpenShift DaemonSet Deployment Specification                    |
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

## Monitor an Application
To deploy an Application that is monitored by the AppDynamics Java Agent you need to do the following:
* Add the Volume to your Application **Adjust accordingly if you do not use hostPath**
* Patch the deployment so we can mount our host volume **Only needed for hostPath**
  * ```oc patch dc <YOUR_APP> -p '{"spec":{"template":{"spec":{"serviceAccountName":"appd"}}}}'```
* Add the Host Volume
  * ```oc volume dc/<YOUR_APP> --add -t hostPath -m /app-agent --path /opt/app-agent/```
* Set some environment variables
  * APPDYNAMICS_AGENT_APPLICATION_NAME - Set AppDynamics App Name
  * APPDYNAMICS_AGENT_TIER_NAME - Set AppDynamics Tier Name
  * JAVA_OPTS - Actually adding the Agent and use Node Reuse
  * ```oc env dc/<YOUR_APP> APPDYNAMICS_AGENT_APPLICATION_NAME=app APPDYNAMICS_AGENT_TIER_NAME=springscale JAVA_OPTS="-Dappdynamics.agent.reuse.nodeName.prefix=springscale -Dappdynamics.agent.reuse.nodeName=true -javaagent:/app-agent/javaagent.jar"```

## Agent Base Image
To deploy the DaemonSet a Docker Base Image is used. This Image can be found in the [Docker Agent Repository].

[Docker Agent Repository]: https://github.com/michaelenglert/docker.appd_agents
