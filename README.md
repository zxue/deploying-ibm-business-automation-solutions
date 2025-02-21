# Deploying IBM Business Automation Solutions

IBM [Cloud Pak for Business Automation](https://www.ibm.com/products/cloud-pak-for-business-automation) (CP4BA) is a modular set of integrated software components, built for any hybrid cloud, designed to automate work and accelerate business growth. Check the documentation on [business automation capabilities](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=overview-automation-capabilities) for more details.

CP4BA versions 24.x are currently available. [CP4BA](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=overview-what-is-cloud-pak-business-automation) can be deployed in kubernetes based containers (OpenShift) or traditional server-based installations. Also, [Cloud Pak for Business Automation as a Service](https://www.ibm.com/docs/en/dbaoc?topic=overview-what-is-cloud-pak-business-automation-as-service) is available.

This document covers CP4BA deployment on OpenShift on-prem or in the cloud.

Note that CP4BA differs from [IBM Automation](https://www.ibm.com/automation), which focuses on AI-powered automation, including IBM iPaaS, Instana, Apptio and Maximo Application Suite.

## Deploying CP4BA using scripts

Prior to CP4BA deployment, you must review [deployment options](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=installing-review-your-deployment-options) and choose one that works best for you. Also, you must consider deployment sizes, non-production vs. production. 

This document covers non-production, starter deployment option. You can find detailed steps in the documentation, [Setting up the cluster with the admin script](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=scripts-setting-up-cluster-admin-script).

### Download the CP4BA installation scripts

You can find the latest version from [IBM Support](https://www.ibm.com/support/pages/node/6576423) site. Clone the github repo locally.

```
 git clone -b 24.0.1 https://github.com/icp4a/cert-kubernetes.git
```

### Log in to OpenShift and create a service account

Follow the [steps](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=deployment-preparing-starter) here.

```
oc login --token=xxx --server=https://api.xxx.com:6443

# cd cp4ba

export NAMESPACE=cp4ba-starter
oc create namespace ${NAMESPACE} 

# apply the yaml file to create the service account

oc apply -f service-account-for-starter.yaml -n ${NAMESPACE}
oc adm policy add-scc-to-user anyuid  -z ibm-cp4ba-anyuid -n ${NAMESPACE}
```

> [!NOTE]  
> - The service account is required and can be created prior to deployment or after. Without the service account, the deployment may appear error free, but the required  LDAP provider is not installed and the CP4BA services are not available.
> - Creation of the non-admin account for OpenShift is optional.

### Run the setup script first

```
# cd cert-kubernetes/scripts
./cp4a-clusteradmin-setup.sh
```

You will need Docker or Podman installed locally. For Podman on Mac, run the podman commands below.

```
#using podman. You will need to create a virtual environment on mac.
podman machine init
podman machine start
```

Make sure that you have IBM license key handy. Choose the options, as shown in the example below. Make a note of the storage classes, which will be used in the next step. Storage classes can also be found in OpenShift directly.

```
Do you wish setup your cluster for a online based CP4BA deployment or for a airgap/offline based CP4BA deployment : 
1) Online
2) Offline/Airgap
Enter a valid option [1 to 2]: 1

Select the cloud platform to deploy: 
1) RedHat OpenShift Kubernetes Service (ROKS) - Public Cloud
2) Openshift Container Platform (OCP) - Private Cloud
3) Other ( Certified Kubernetes Cloud Platform / CNCF)
Enter a valid option [1 to 3]: 2

What type of deployment is being performed?
1) Starter
2) Production
Enter a valid option [1 to 2]: 1

[NOTES] You can install the CP4BA deployment as either a private catalog (namespace scope) or the global catalog namespace (GCN). The private option uses the same target namespace of the CP4BA deployment, the GCN uses the openshift-marketplace namespace.
Do you want to deploy CP4BA using private catalog (recommended)? (Yes/No, default: Yes): 

Where do you want to deploy Cloud Pak for Business Automation?
Enter the name for a new project or an existing project (namespace): cp4ba-starter
```

### Run the deployment script next

```
# cd cert-kubernetes/scripts
./cp4a-deployment.sh -n ${NAMESPACE}
```

Choose one or multiple CP4BA capabilities. For example, type 2, press the enter/return key. Repeat the two steps. Once done, press the enter/return key.

```
Select the Cloud Pak for Business Automation capability to install: 
1) FileNet Content Manager 
2) Operational Decision Manager (Selected)
3) Automation Decision Services 
4) Business Automation Application (Selected)
5) Business Automation Workflow Authoring and Automation Workstream Services (Selected)
6) IBM Automation Document Processing 

Info: Except pattern (4/5), Business Automation Navigator will be automatically installed in the environment as it is part of the Cloud Pak for Business Automation foundation platform. 

Tips:  After you make your first selection you will be able to make additional selections since you can combine multiple selections.

[ATTENTION]: IBM Automation Document Processing (6) does NOT support a cluster running a Linux on Z (s390x)/Power architecture.

Tips:Press [ENTER] when you are done

Pattern "Operational Decision Manager": Select optional components: 
1) Business Automation Insights (Selected)

[ATTENTION]:  You must select at least one of ODM components.

Tips: Decision Center, Rule Execution Server and Decision Runner will be installed by default.

Pattern "Business Automation Workflow Authoring and Automation Workstream Services": Select optional components: 
1) Case 
2) Content Integration 
3) Workstreams 
4) Data Collector and Data Indexer 
5) Business Automation Insights (Selected)
6) Business Automation Machine Learning 

Tips: Press [ENTER] if you do not want any optional components or when you are finished selecting your optional components
Enter a valid option [1 to 6 or ENTER]: 
```

Provide the storage classes for read write many (WRX) and read write once (WRO). See example below.

```
To provision the persistent volumes and volume claims, enter the file storage classname(RWX): ocs-storagecluster-cephfs
To provision the persistent volumes and volume claims, enter the block storage classname(RWO): ocs-storagecluster-ceph-rbd

[INFO] For starter deployment, always setting "sc_restricted_internet_access" as "true" in final custom resource.

Installing the selected Cloud Pak capability...
icp4acluster.icp4a.ibm.com/icp4adeploy created
Done
The custom resource file located at: "/Users/benjamin/cp4ba/cert-kubernetes/scripts/generated-cr/project/cp4ba-starter/ibm_cp4a_cr_final.yaml"

To monitor the deployment status, follow the Operator logs.
For details, refer to the troubleshooting section in Knowledge Center here: 
https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=automation-troubleshooting
```

### Validate CP4BA starter deployment

The overall CP4BA deployment experience has been improved tremendously by the IBM team. Depending on what capabilities you have selected, the deployment may take some time. To check the deployment status, follow the [steps](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=scripts-validating-your-starter-deployment) here.

```
# cd cert-kubernetes/scripts

# deployment instance name e.g. icp4adeploy
oc get icp4acluster
oc describe icp4acluster icp4adeploy -n ${NAMESPACE} 
oc get ICP4ACluster icp4adeploy -o=jsonpath='{.status.components.baw}'
oc get ICP4ACluster icp4adeploy -o=jsonpath='{.status.components.odm}'

export CP4BA_COMMON_SERVICES_NAMESPACE=${NAMESPACE}
./cp4a-post-install.sh --precheck
./cp4a-post-install.sh --starterStatus
```

You can find a sample output for the starter deployment below.

```
CP4BA Service Status - High level
################################################################
CP4BA Version                               :  24.0.1 
Project/Namespace                           :  cp4ba
Zen Version                                 :  NotInstalled
Message 1                                   :  Running reconciliation
Message 2                                   :  
Deployment Type                             :  Starter 
Status Dump                                 :  /Users/benjamin/cp4ba/cert-kubernetes/scripts/logs/icp4adeploy-cp4ba-status-info.yaml

CP4BA Service Status - Insights
################################################################
bai_deploy_status:                          :  NotInstalled
insightsEngine:                             :  NotInstalled

CP4BA Service Status - BA Studio
################################################################
service:                                    :  NotInstalled

CP4BA Service Status - Task Manager 
################################################################
tmDeployment                                :  NotInstalled
tmService                                   :  NotInstalled
tmRoute                                     :  NotInstalled
tmStorage                                   :  NotInstalled

CP4BA Service Status - Navigator
################################################################
navigatorDeployment                         :  NotInstalled
navigatorService                            :  NotInstalled
navigatorStorage                            :  NotInstalled
navigatorZenIntegration                     :  NotInstalled

CP4BA Service Status - Workflow
################################################################

CP4BA Service Status - BAML
################################################################
bamlCustomResource                          :  NotInstalled

CP4BA Service Status - PFS
################################################################
pfsDeployment                               :  NotInstalled
pfsService                                  :  NotInstalled
pfsZenIntegration                           :  NotInstalled

CP4BA Service Status - Business Automation Application
################################################################
workspace-aae service                       :  NotInstalled
pbk service                                 :  NotInstalled

CP4BA Service Status - Automation Decision Services
################################################################
adsCredentialsServiceDeployment             :  NotInstalled
adsCredentialsServiceService                :  NotInstalled
adsDownloadServiceDeployment                :  NotInstalled
adsDownloadServiceService                   :  NotInstalled
adsFrontDeployment                          :  NotInstalled
adsFrontZenIntegration                      :  NotInstalled
adsGitServiceService                        :  NotInstalled
adsLtpaCreationJob                          :  NotInstalled
adsMongoService                             :  NotInstalled
adsParsingServiceService                    :  NotInstalled
adsRestApiService                           :  NotInstalled
adsRuntimeBaiRegistrationJob                :  NotInstalled
adsRunServiceService                        :  NotInstalled
adsRuntimeBaiRegistrationJob                :  NotInstalled
adsRuntimeServiceService                    :  NotInstalled

CP4BA Service Status - Operational Decision Manager
################################################################
odmDecisionCenterDeployment                 :  NotInstalled
odmDecisionCenterService                    :  NotInstalled
odmDecisionCenterZenIntegration             :  NotInstalled
odmDecisionRunnerDeployment                 :  NotInstalled
odmDecisionRunnerService                    :  NotInstalled
odmDecisionRunnerZenIntegration             :  NotInstalled
odmDecisionServerConsoleDeployment          :  NotInstalled
odmDecisionServerConsoleService             :  NotInstalled
odmDecisionServerConsoleZenIntegration      :  NotInstalled
odmDecisionServerRuntimeDeployment          :  NotInstalled
odmDecisionServerRuntimeService             :  NotInstalled
odmDecisionServerRuntimeZenIntegration      :  NotInstalled

CP4BA Service Status - Content
################################################################
cpeDeployment                               :  NotInstalled
cpeJDBCDriver                               :  NotInstalled
cpeRoute                                    :  NotInstalled
cpeService                                  :  NotInstalled
cpeStorage                                  :  NotInstalled
cpeZenIntegration                           :  NotInstalled
cmisDeployment                              :  NotInstalled
cmisRoute                                   :  NotInstalled
cmisService                                 :  NotInstalled
cmisStorage                                 :  NotInstalled
cmisZenIntegration                          :  NotInstalled
ierDeployment                               :  NotInstalled
ierRoute                                    :  NotInstalled
ierService                                  :  NotInstalled
ierStorageCheck                             :  NotInstalled
graphqlDeployment                           :  NotInstalled
graphqlRoute                                :  NotInstalled
graphqlService                              :  NotInstalled
graphqlStorage                              :  NotInstalled

```

### Find CP4BA admins and users

A successful deployment creates several [admin user accounts](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=scripts-installing-capabilities-by-running-deployment-script). 
You can log in to the CP4BA cluster in one of the options below.

- Use OpenShift admin credentials: kubeadmin, provided password
- Use BM provided credentials (cpadmin only): cpadmin and password. See command below.
- Use Enterprise LDAP, cp4admin and password. The default user credentials are stored in the secret named "icp4adeploy-openldap-customldif".

```
# get cpadmin password
oc -n <namespace> get secret platform-auth-idp-credentials -o jsonpath='{.data.admin_password}' | base64 -d && echo
```

Many routes are created in OpenShift, two of which are of particular interest to us. One starts with "cp-console" and another with "cpd", as shown below.

```
https://cp-console-<cp4ba namespace>.apps.xxx.com/
https://cpd-<cp4ba namespace>.apps.xxx.com
```

## Deleting CP4BA deployment

Check out the documentation on [Uninstalling capabilities from the OCP console](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=capabilities-uninstalling-from-ocp-console).

To delete a testing CP4BA deployment manually, you can delete the CP4BA project in OpenShift. However, you may notice that the project stays in "Terminating" state. 

Open the yaml file for the project. You see something like this.

```
message: 'Some resources are remaining: 
authentications.operator.ibm.com has 1 resource instances, 
clients.oidc.security.ibm.com has 1 resource instances, 
operandbindinfos.operator.ibm.com has 3 resource instances, 
operandrequests.operator.ibm.com has 8 resource instances, 
zenextensions.zen.cpd.ibm.com has 1 resource instances'
    - type: NamespaceFinalizersRemaining
      status: 'True'
      lastTransitionTime: '2025-02-19T18:15:57Z'
      reason: SomeFinalizersRemain
      message: 'Some content in the namespace has finalizers remaining: 
authentication.operator.ibm.com in 1 resource instances, 
client.oidc.security.ibm.com in 1 resource instances, 
finalizer.bindinfo.ibm.com in 3 resource instances, 
finalizer.request.ibm.com in 8 resource instances, 
zenextension.zen.cpd.ibm.com/finalizer in 1 resource instances'
```

Go to CustomResourceDefinitions under Administration in OpenShift. Locate the CR resource for each, and click on the Instances tab. You will notice one or more resources listed in the CP4BA namespace. Click and open the yaml file for each resource, delete the following two lines and save the yaml file. 

```
  finalizers:
    - finalizer.request.ibm.com
```

Once all resources are cleaned up, the CP4BA project is then deleted and a new one with the same or different namespace can be created.

## Troubleshoot deployment issues

Check out the [troubleshooting](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=installing-troubleshooting) documentation.

### Missing Enterprise LDAP login option

When a CP4BA deployment is partially finished or fails, the login option is not available. One of several possible reasons is that the service account is missing or that permissions are not granted to the service account. 

Fix: create the service account and apply permissions to it.

### The CP4BA deployment fails

One possible reason is that incorrect storage classes are used for rwx and rwo storages.

Fix: delete the project and re-deploy it with the correct options, including the right storage classes.

### Some PVCs are in pending state in OpenShift

One possible reason is that image registry storage is not configured, for example, in a VMware environment. Follow the [documentation](https://docs.openshift.com/container-platform/4.15/registry/configuring_registry_storage/configuring-registry-storage-vsphere.html) to configure the storage.

- Run the command line tocChange managementState Image Registry Operator configuration from Removed to Managed.
```
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
```

- Run the commandline to modify the configuration
```
oc edit configs.imageregistry.operator.openshift.io
```

Update the storage spec as follows and save the change.
```
storage:
  pvc:
    claim: 
```

- Check the pvc for image registry operator. If it is in pending, re-create it with the correct storage. See one example below.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: image-registry-storage
  namespace: openshift-image-registry
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: ocs-external-storagecluster-cephfs
  volumeMode: Filesystem
```

Another possible reason is that some PVCs are in pending, for example, `operator-shared-pvc`(1 GiB, rwx) and `opensearch-ib-6fb9-es-server-snap` (5 GiB, rwx).

Fix: delete the PVCs and re-create them with the correct names in the CP4BA namespace.

### The icp-4adeploy-openldap-deploy-xxx pod keeps crashing

One possible reason is that the service account permission is not granted properly. For example, the "\" in the command line might have caused the problem. Removing the extra character appeared to be a fix to the issue. However, there may be other reasons that require further investigation.

```
oc adm policy add-scc-to-user anyuid \ -z ibm-cp4ba-anyuid -n ${NAMESPACE}
```

### Slow performance issues

The [starter deployment](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=deployment-system-requirements) requires 3 minimum worker nodes, whereas [small, medium or large deployments](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/24.0.1?topic=pcmppd-system-requirements) require 8 or more worker nodes.

Fix: Increase the number of worker nodes may help improve overall service performance.

In scenarios where, for example, a CP4BA is deployed in the U.S. and developers are based in other regions like Asia Pacific, users may experience network latency, which can cause screen refresh intermittently and connection to server timeouts briefly. 

One workaround is to provision end user or developer VMs in the same region where the CP4BA is deployed. Optionally, the VMs may be provisioned in the cloud region close to the user or developer. 


## Acknowledgement

Thanks to my IBMers, Alex Cravalho and Matt Womack for their review and feedback.




