# A little bit of security is what I want

Segregation of duties - how a Kubernetes Management Platform add security to your roll outs

## Starting point

Based on Stefan's post [Configuring custom domains for applications – the operator-way](https://www.opensourcerers.org/2022/12/12/configuring-custom-domains-for-applications-the-operator-way/) in December some of us felt inspired to replay what Stefan outlined. Main topics we wanted to cover were
* Utilize a custom domain name for a project
* Utilize a K8s management platform
* Leave all certificate handling with the K8s platform
* Be able to create new projects leveraging the certification and DNS handling with a minimum effort


## 

The below picture illustrates the flow of things in our Red Hat Openshift environment
+ Roles
  + OpenShift Administrator [has administrative rights in a cluster (aka cluster admin)]
  + OpenShift Developer [Has only limited rights which are mostly restricted to our project]
+ Instances
  + OpenShift [Kubernetes Management Platform]
  + Certificate Authority [CA - [we used Let's Encrypt](https://en.wikipedia.org/wiki/Let's_Encrypt)]


![dns_sequence_overview](images/winkelschleifer-sequence.png)



## References

[1] [Configuring custom domains for applications – the operator-way](https://www.opensourcerers.org/2022/12/12/configuring-custom-domains-for-applications-the-operator-way/)

## Images


![dns_sequence_admin](images/winkelschleifer-sequence-admin.png)

![dns_sequence_developer](images/winkelschleifer-sequence-Developer2.png)