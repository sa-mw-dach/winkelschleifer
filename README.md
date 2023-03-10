# winkelschleifer
Google  microservice addition

## Deploy microservices-demo on ROSA
---
### Prepare ROSA environment
---
Install CLI Tools, e.g. on macos via homebrew:

* brew install openshift-cli
* brew install aws-cli
* brew install rosa-cli

### Set up ROSA 
---
→ follow the console.redhat.com instructions

```
rosa create cluster --cluster-name winkelschleifer --sts --role-arn arn:aws:iam::226507396508:role/ManagedOpenShift-Installer-Role --support-role-arn arn:aws:iam::226507396508:role/ManagedOpenShift-Support-Role --controlplane-iam-role arn:aws:iam::226507396508:role/ManagedOpenShift-ControlPlane-Role --worker-iam-role arn:aws:iam::226507396508:role/ManagedOpenShift-Worker-Role --operator-roles-prefix winkelschleifer-s5c8 --region eu-central-1 --version 4.11.18 --compute-nodes 2 --compute-machine-type m5.xlarge --machine-cidr 10.0.0.0/16 --service-cidr 172.30.0.0/16 --pod-cidr 10.128.0.0/14 --host-prefix 23 --etcd-encryption
```

Check the cluster status on console.redhat.com
You might need to add some details manually:

![image_rosa_cli](images/winkelschleifer_010.png)

**ROSA installation:** https://console-openshift-console.apps.winkelschleifer.wbcq.p1.openshiftapps.com

## Configure Google Auth
---
https://examples.openshift.pub/cluster-configuration/authentication/redhat-sso/  


### Create a new project
---

![image_google_auth](images/winkelschleifer_011.png)

![image_google_auth](images/winkelschleifer_013.png)

### Add OAuth Client Ids
---
![image_google_auth](images/winkelschleifer_012.png)

![image_google_auth](images/winkelschleifer_014.png)

![image_google_auth](images/winkelschleifer_015.png)

![image_google_auth](images/winkelschleifer_016.png)

## Add Google OAuth as identity provider
---

```
rosa create idp --type=google --client-id=626307336472-mrtf4suln41vs30oop1gbb92qmlb57it.apps.googleusercontent.com --client-secret=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 --mapping-method=claim --hosted-domain=redhat.com --cluster=winkelschleifer --name=Google-RedHat
```

### Add Users to dedicated admin role
---
Add dedicated users which login via Google ID to the local group of cluster admins

```
rosa grant user dedicated-admin --user=wrichter@redhat.com --cluster=winkelschleifer
```

# Deploy Application
---
Locally clone the Google project

```
git clone https://github.com/GoogleCloudPlatform/microservices-demo
```

## Create new project in ROSA
---
The Red Hat OpenShift cluster you create here will be base of our following activities

```
oc new-project microservices-demo
```

## Deploy Application
---

This is done in order to execute a deployment of Google Code As-Is. Current deployments use the following security contexts - we need to make arrangements that those settings are valid in our cluster too.

To be found in microservices-demo/release/kubernetes-manifest.yaml

```
securityContext:
        fsGroup: 1000
        runAsGroup: 1000
        runAsNonRoot: true
        runAsUser: 1000
```

Based on this article: https://examples.openshift.pub/deploy/scc-anyuid/ 

### Option 1: Give ANYUID permissions to default service account
---

Add the proper Role
```
oc create -f static/release_security_role.json
```
Add the related RoleBinding
```
oc create -f static/release_security_rolebonding.json
```
And use the original kubernetes manifest

```
oc apply -f ./release/kubernetes-manifests.yaml
```

### Option 2: Remove SecurityContext & Deploy application
---

This makes the default OpenShift SCC config work. And we use the As-Is Kubernetes Manifest:

```
oc apply -f ./release/kubernetes-manifests.yaml
```
And we run through each of the deploaments in order to fix the security context(s)

```
for DEPLOY in `oc get deployments -o jsonpath='{range .items[*]}{.metadata.name}{" "}{end}'`
do
 oc patch deploy/$DEPLOY --type=json -p='[{"op": "remove", "path": "/spec/template/spec/securityContext"}]'
done
```

# Overview of the installation flow
---

And following Monty Python's Flying Circus .... [And now for something completely  different](https://en.wikipedia.org/wiki/And_Now_for_Something_Completely_Different)

![image_google_auth](images/winkelschleifer-sequence.png)
This picture illustrates the flow of things in our Red Hat Openshift environment
+ Roles
  + OpenShift Administrator [has administrative rights in a cluster (aka cluster admin)]
  + OpenShift Developer [Has only limited rights which are mostly restricted to our project]
+ Instances
  + OpenShift [Kubernetes Management Platform]
  + Certificate Authority [CA - [we used Let's Encrypt](https://en.wikipedia.org/wiki/Let's_Encrypt)]
  + DNS Zone [Administrative access to manage on subdomain level]
+ Tasks
  + Configuration [Admin tasks prior to the instalation]
  + ExternalDNS [Tasks executed by the ExternalDNS Operator]
  + Cert-Manager [Tasks executed by the cert-manager Operator]
+ Usage [Separated block of activities which are executed and orchestrated by the operators during runtime]


## 1 Install Operators and basic configuration(s)
---

( ExternalDNS, cert-manager, cert-utils)

We see the default Loadbalancers which are deployed by EKS

![image_AWS_LoBa_view](images/AWS_LoBa_010.png)

## 2 Create DNS hosted zone
---

![image_AWS_Route53](images/AWS_Route53_HostedZone.png)

![image_AWS_wildcard](images/AWS_WildcardCNAME.png)

You will need to add nameserver information in the config in your provider's frontend

![image_DNS_NS_entried](images/DNS_Create_NS_entries.png)

And we can now add a Red Hat OpenShift route to your service

```
oc create route edge winkelschleifer-ingress \
--service frontend --port 8080 \
--namespace microservices-demo \
--hostname shop.winkelschleifer.marina-sport.de \
--insecure-policy=”redirect”
```

This leads us to the point that we have a route which terminates at our frontend service

![image_route_frontend](images/winkelschleifer_043.png)

And this can be checked as per below command

```
oc get route
```
```
NAME                      HOST/PORT                                PATH   SERVICES   PORT   TERMINATION     WILDCARD
testit-tls                testit.winkelschleifer.marina-sport.de          frontend   8080   edge            None
winkelschleifer-ingress   shop.winkelschleifer.marina-sport.de            frontend   8080   edge/Redirect   None
```


## 3 Create DNS API access credentials
---

## 4 Create CA access credentials
---

## ExternalDNS config
---

![image_Operator_ExternalDNS](images/Operator_ExternalDNS.png)

### 5 create secret with DNS access credentials
---

### 6 create ExternalDNS configuration
---

## Cert-Manager
---

![image_Operator_certManager](images/Operator_certManager.png)

### 7 create secret with DNS access credentials
---

### 8 create secret with CA access credentials
---

### 9 create cert-manager Clusteruser configuration
---

