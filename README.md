# winkelschleifer
Google  microservice addition

# winkelschleifer
Google  microservice addition

## Deploy microservices-demo on ROSA
### Prepare ROSA environment
Install CLI Tools, e.g. on macos via homebrew:

* brew install openshift-cli
* brew install aws-cli
* brew install rosa-cli

### Set up ROSA 
→ follow the console.redhat.com instructions

```
rosa create cluster --cluster-name winkelschleifer --sts --role-arn arn:aws:iam::226507396508:role/ManagedOpenShift-Installer-Role --support-role-arn arn:aws:iam::226507396508:role/ManagedOpenShift-Support-Role --controlplane-iam-role arn:aws:iam::226507396508:role/ManagedOpenShift-ControlPlane-Role --worker-iam-role arn:aws:iam::226507396508:role/ManagedOpenShift-Worker-Role --operator-roles-prefix winkelschleifer-s5c8 --region eu-central-1 --version 4.11.18 --compute-nodes 2 --compute-machine-type m5.xlarge --machine-cidr 10.0.0.0/16 --service-cidr 172.30.0.0/16 --pod-cidr 10.128.0.0/14 --host-prefix 23 --etcd-encryption
```

Check the cluster status on console.redhat.com
You might need to add some details manually:

![image_rosa_cli](images/winkelschleifer_010.png)

**ROSA installation:** https://console-openshift-console.apps.winkelschleifer.wbcq.p1.openshiftapps.com

## Configure Google Auth
https://examples.openshift.pub/cluster-configuration/authentication/redhat-sso/  


### Create a new project

![image_google_auth](images/winkelschleifer_011.png)

![image_google_auth](images/winkelschleifer_013.png)

### Add OAuth Client Ids
![image_google_auth](images/winkelschleifer_012.png)

![image_google_auth](images/winkelschleifer_014.png)

![image_google_auth](images/winkelschleifer_015.png)

![image_google_auth](images/winkelschleifer_016.png)

## Add Google OAuth as identity provider

```
rosa create idp --type=google --client-id=626307336472-mrtf4suln41vs30oop1gbb92qmlb57it.apps.googleusercontent.com --client-secret=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 --mapping-method=claim --hosted-domain=redhat.com --cluster=winkelschleifer --name=Google-RedHat
```

### Add Users to dedicated admin role
Add dedicated users which login via Google ID to the local group of cluster admins

```
rosa grant user dedicated-admin --user=wrichter@redhat.com --cluster=winkelschleifer
```

# Deploy Application
Locally clone the Google project

```
git clone https://github.com/GoogleCloudPlatform/microservices-demo
```

## Create new project in ROSA
The Red Hat OpenShift cluster you create here will be base of our following activities

```
oc new-project microservices-demo
```

## Deploy Application
### Option 1: Give ANYUID permissions to default service account

This is done in order to execute a deployment of Google Code As-Is. Current deployments use the following security contexts - we need to make arrangements that those settings are valid in our cluster too.

To be found in microservices-demo/release/kubernetes-manifest.yaml

```
securityContext:
        fsGroup: 1000
        runAsGroup: 1000
        runAsNonRoot: true
        runAsUser: 1000
```




**DNS - wer macht hier eigentlich was**

![image_google_auth](images/winkelschleifer_020.jpg)


