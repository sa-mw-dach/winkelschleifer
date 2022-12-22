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
