# A little bit of security is what I want

Segregation of duties - how a Kubernetes Management Platform adds security to your roll outs

## Introduction

Based on Stefan's post [1] **Configuring custom domains for applications – the operator-way** in December, some of us felt inspired to replay what Stefan outlined.

Main topics we wanted to cover:
* Utilize a custom domain name for a project
* Utilize a Kubernetes management platform [K8s]
* Leave all certificate handling with the K8s platform
* Be able to create new projects leveraging the certification and DNS handling with a minimum effort

And as Red Hat employees the K8s management platform choice is more than obvious. We decided for Red Hat OpenShift as a base for the next steps.

The concept of operators (see [2]) is used and referenced in Stefan's post. And while running through the described topics you will see that operators and the K8s management platform solve security aspects in a standard workflow.

## Starting point

We started with the below workflow

1. Deploy a managed Red Hat OpenShift platform in either AWS or Azure or Google
2. Configure an authentication provider to utilize the on-board multi tenant capabilites of Red Hat OpenShift
3. Install the three operators ( cert utils, external DNS managament and cert manager)
4. Deploy the example application
5. Expose the application with a secure, custom domain name

And here we made a change to the details in [1], when we started to utilize a domain hosted "somewhere". In order to do so we had to ask for help from the outside.

**At this point we started to discuss security aspects of our workflow**

<img src="images/hl-architecture-change.png" align="center" alt="high level architecture change" width="200"/> </p>

Delegating such tasks to an operator instead of asking for help from the outside of your team adds security and speeds up deployments.


## The Operators

---

The operators we will use are listed below

[//]:![cert-utils-operator](images/cert-utils-op.png)
[//]:![cert-utils-operator](images/cert-manager.png)
[//]:![cert-utils-operator](images/ext-dns-op.png)

<img src="images/cert-utils-op.png" alt="cert-utils-operator" width="200"/>
<img src="images/cert-manager.png" alt="cert-manager-operator" width="200"/>
<img src="images/ext-dns-op.png" alt="external-dns-operator" width="200"/>

And please keep in mind that operators do a lot more for your deployments, as they add operational knowledge to your software ( see [2])

---

## The Workflow

We drew a picture to make ourselves aware of the roles, the involved instances, and the operator based workflow.


The below picture illustrates the flow of things in our Red Hat Openshift environment
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

![dns_sequence_overview](images/winkelschleifer-sequence.png)

## How security plays along

There are two major blocks of activities which need to happen.

First - the **administrative ones** which need to be executed with additional rights. These tasks are mostly handled outside of application teams. Respresented by the **red box**.

<img src="images/winkelschleifer-sequence-admin.png" alt="Admin tasks" width="300"/>

And there are many tasks to be done in order to configure the operators right. All these tasks have impact on systems outside of the developer's responsiblity. Represented by the **blue box**

<img src="images/winkelschleifer-sequence-admin-tasks.png" alt="Admin tasks" width="300"/>

To add security to your workflows you want to keep these tasks outside of the development team's responsibility. And at the same time the deployment frequency should stay high or ideally become higher. 

Second - the **application operational ones** which are executed by application development and application operations teams in typical enterprise environments. Represented by the **green box**.

<img src="images/winkelschleifer-sequence-Developer2.png" alt="Developer tasks" width="300"/>

To achieve high deployment frequency and still stay secure the development teams benefit from the configured operators, because:
* **yellow box** all tasks which have dependencies to systems outside of the application are executed by the operators
* **blue box** there is a single task which is initiated by the development team and utilizing the operator initiated tasks

<img src="images/winkelschleifer-sequence-Developer-task.png" alt="Developer tasks" width="300"/>


## Conclusion

A Kubernetes Management platform and operators help you in two ways, when it comes to security

**First** - support to secure your applications by making sure that end-to-end encryption is in place, is easily manageable, includes secure management of external resources (DNS and CA)

**Second** - and this is an often underestimated puzzle part in K8s management - it supports to live segregation of duties. Development teams are enabled to roll out new applications "by click of a button" and at the same time the DNS management is kept separate to ensure security.

And this is a lot of security you get !

Happy 2023 !!

## References

[1] [Configuring custom domains for applications – the operator-way](https://www.opensourcerers.org/2022/12/12/configuring-custom-domains-for-applications-the-operator-way/)

[2] [The Operator Framework](https://operatorframework.io/)