# Automated VMware Cloud Foundation Lab Deployment

## Table of Contents

* [Description](#description)
* [Changelog](#changelog)
* [Requirements](#requirements)
* [Management Domain Configuration](#management-domain-configuration)
* [Workload Domain Configuration](#workload-domain-configuration)
* [Logging](#logging)
* [Sample Execution](#sample-execution)
    * [Deploy Nested ESXi and Cloud Builder VMs](#deploy-nested-esxi-and-cloud-builder-vms)
    * [Deploy VCF Management Domain](#deploy-vcf-management-domain)
    * [Deploy VCF Workload Domain](#deploy-vcf-workload-domain)

## Description

Similar to previous "Automated Lab Deployment Scripts" (such as [here](https://www.williamlam.com/2016/11/vghetto-automated-vsphere-lab-deployment-for-vsphere-6-0u2-vsphere-6-5.html), [here](https://www.williamlam.com/2017/10/vghetto-automated-nsx-t-2-0-lab-deployment.html), [here](https://www.williamlam.com/2018/06/vghetto-automated-pivotal-container-service-pks-lab-deployment.html), [here](https://www.williamlam.com/2020/04/automated-vsphere-7-and-vsphere-with-kubernetes-lab-deployment-script.html), [here](https://www.williamlam.com/2020/10/automated-vsphere-with-tanzu-lab-deployment-script.html) and [here](https://williamlam.com/2021/04/automated-lab-deployment-script-for-vsphere-with-tanzu-using-nsx-advanced-load-balancer-nsx-alb.html)), this script makes it very easy for anyone to deploy a "basic" VMware Cloud Foundation (VCF) in a Nested Lab environment for learning and educational purposes. All required VMware components (ESXi and Cloud Builder VMs) are automatically deployed and configured to allow for VCF to be deployed and configured using VMware Cloud Builder. For more information, you can refer to the official [VMware Cloud Foundation documentation](https://techdocs.broadcom.com/us/en/vmware-cis/vcf.html).

Below is a diagram of what is deployed as part of the solution and you simply need to have an existing vSphere environment running that is managed by vCenter Server and with enough resources (CPU, Memory and Storage) to deploy this "Nested" lab. For VCF enablement (post-deployment operation), please have a look at the [Sample Execution](#sample-execution) section below.

You are now ready to get your VCF on! 😁

![](screenshots/screenshot-0.png)

## Changelog
* **02/28/2025**
  * Externalize user variables to configuration files
  * Fixed Workload Domain spec construction for vLCM-based deployment
  * Add support for VCF 5.2.1.1
* **10/09/2024**
  * Add support for VCF 5.2.1
    * NSX Manager size changed from `small` to `medium` (needed for 5.2.1 or have seen deployment issues)
* **07/10/2024**
  * Management Domain:
    * Add support for VCF 5.2 (password for Cloud Builder 5.2 must be 15 characters minimum)
  * Workload Domain:
    * Add support for VCF 5.2
    * Add `$SeparateNSXSwitch` variable to specify separate VDS for NSX (simliar to Management Domain option)
* **05/28/2024**
  * Management Domain:
    * Refactor VCF Management Domain JSON generation to be more dynamic
    * Refactor licensing code to support both licensed keys or license later feature
    * Add required `clusterImageEnabled` to JSON by default using `$EnableVCLM` variable
  * Workload Domain:
    * Add `$EnableVCLM` variable to control vLCM-based image for vSphere Cluster
    * Add `$VLCMImageName` variable to specify desired vLCM-based image (default uses Management Domain)
    * Add `$EnableVSANESA` variable to specify whether vSAN ESA is enabled
    * Add `$NestedESXiWLDVSANESA` variable to specify whether Nested ESXi VM for WLD will be used for vSAN ESA, requiring NVME controller vs PVSCSI controller (default)
    * Refactor licensing code to support both licensed keys or license later feature
* **03/27/2024**
  * Added support for license later (aka 60 day evaluation mode)
* **02/08/2024**
  * Added supplemental script `vcf-automated-workload-domain-deployment.ps1` to automate the deployment of Workload Domain
* **02/05/2024**
  * Improve substitution code for ESXi vMotion, vSAN & NSX CIDR network
  * Renamed variables (`$CloudbuilderVMName`,`$CloudbuilderHostname`,`$SddcManagerName`,`$NSXManagerVIPName`,`$NSXManagerNode1Name`) to (`$CloudbuilderVMHostname`,`$CloudbuilderFQDN`,`$SddcManagerHostname`,`$NSXManagerVIPHostname`,`$NSXManagerNode1Hostname`) to better represent the expected value (Hostname and FQDN)
* **02/03/2024**
  * Added support to independently define resources (cpu, memory and storage) for Nested ESXi VMs for use with Management and/or Workload Domains
  * Automatically generate VCF Workload Domain host commission JSON file (vcf-commission-host-api.json) for use with SDDC Manager API (UI will now include `-ui` in the filename)
* **01/29/2024**
  * Added support for [VCF 5.1]([text](https://blogs.vmware.com/cloud-foundation/2023/11/07/announcing-availability-of-vmware-cloud-foundation-5-1/))
  * Automatically start VCF Management Domain bringup in SDDC Manager using generated JSON deployment file (vcf-mgmt.json)
  * Added support for deploying Nested ESXi hosts for Workload Domain
  * Automatically generate VCF Workload Domain host commission JSON file (vcf-commission-host.json) for SDDC Manager
  * Added `-CoresPerSocket` argument to optimize for Nested ESXi deployment for licensing
  * Added variables (`$NestedESXivMotionNetworkCidr`, `$NestedESXivSANNetworkCidr` and `$NestedESXiNSXTepNetworkCidr`) for customizing ESXi vMotion, vSAN and NSX TEP network CIDRs

* **03/27/2023**
  * Enable Multiple deployment on the same Cluster

* **02/28/2023**
  * Added note on DRS-enabled Cluster for vApp creation and pre-check in code

* **02/21/2023**
  * Added note to Configuration for deploying VCF Management Domain using only a single ESXi host

* **02/09/2023**
  * Update ESXi Memory to fix "Configure NSX-T Data Center Transport Node" and "Reconfigure vSphere High Availability" failing tasks by increasing ESXi memory to 46GB [explained here](http://strivevirtually.net)

* **01/21/2023**
  * Added support for [VCF 4.5](https://imthiyaz.cloud/automated-vcf-deployment-script-with-nested-esxi)
  * Fixed vSAN bootdisk size
  * Follow [KB 89990](https://knowledge.broadcom.com/external/article?legacyId=89990) if you get "Gateway IP Address for Management is not contactable"
  * If Failed VSAN Diskgroup follow [FakeSCSIReservations](https://williamlam.com/2013/11/how-to-run-nested-esxi-on-top-of-vsan.html)

* **05/25/2021**
  * Initial Release

## Requirements

* Supported VCF Versions and required build-of-materials (BOM)

| VCF Version | Cloud Builder Download                                                                                                                                                                                                                     | Nested ESXi Download                                                       |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| 5.2.1.1     | [ VMware Cloud Builder 5.2.1.1 (24307856) OVA ]([https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Cloud%20Foundation&displayGroup=VMware%20Cloud%20Foundation%205.2&release=5.2.1&os=&servicePk=523724&language=EN)     | [ Nested ESXi 8.0 Update 3c OVA ]( https://community.broadcom.com/flings)  |
| 5.2.1       | [ VMware Cloud Builder 5.2.1 (523724) OVA ]([https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Cloud%20Foundation&displayGroup=VMware%20Cloud%20Foundation%205.2&release=5.2.1&os=&servicePk=523724&language=EN)     | [ Nested ESXi 8.0 Update 3 OVA ]( https://community.broadcom.com/flings)  |
| 5.2         | [ VMware Cloud Builder 5.2 (520823) OVA ]([https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Cloud%20Foundation&displayGroup=VMware%20Cloud%20Foundation%205.2&release=5.2&os=&servicePk=520823&language=EN)     | [ Nested ESXi 8.0 Update 3 OVA ]( https://community.broadcom.com/flings)  |
| 5.1.1       | [ VMware Cloud Builder 5.1.1 (23480823) OVA ]( [https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Cloud%20Foundation&displayGroup=VMware%20Cloud%20Foundation%205.1&release=5.1.1&os=&servicePk=208634&language=EN) | [ Nested ESXi 8.0 Update 2b OVA ]( https://community.broadcom.com/flings) |
| 5.1         | [VMware Cloud Builder 5.1 (22688368) OVA]([https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Cloud%20Foundation&displayGroup=VMware%20Cloud%20Foundation%205.1&release=5.1&os=&servicePk=203383&language=EN)         | [ Nested ESXi 8.0 Update 2 OVA ]( https://community.broadcom.com/flings)  |

* vCenter Server running at least vSphere 7.0 or later
    * If your physical storage is vSAN, please ensure you've applied the following setting as mentioned [here](https://www.williamlam.com/2013/11/how-to-run-nested-esxi-on-top-of-vsan.html)
* ESXi Networking
  * Enable either [MAC Learning](https://williamlam.com/2018/04/native-mac-learning-in-vsphere-6-7-removes-the-need-for-promiscuous-mode-for-nested-esxi.html) or [Promiscuous Mode](https://knowledge.broadcom.com/external/article?legacyId=1004099) and also enable Forged transmits on your physical ESXi host networking to ensure proper network connectivity for Nested ESXi workloads
* Resource Requirements
    * Compute
        * Ability to provision VMs with up to 8 vCPU (12 vCPU required for Workload Domain deployment)
        * Ability to provision up to 384 GB of memory
        * DRS-enabled Cluster (not required but vApp creation will not be possible)
    * Network
        * 1 x Standard or Distributed Portgroup (routable) to deploy all VMs (VCSA, NSX-T Manager & NSX-T Edge)
           * 13 x IP Addresses for Cloud Builder, SDDC Manager, VCSA, ESXi and NSX-T VMs
           * 9 x IP Addresses for Workload Domain Deployment (if applicable) for ESXi, NSX and VCSA
    * Storage
        * Ability to provision up to 1.25 TB of storage

        **Note:** For detailed requirements, plesae refer to the planning and preparation workbook [here](https://techdocs.broadcom.com/us/en/vmware-cis/vcf/vcf-5-2-and-earlier/5-2/planning-and-preparation-workbook-5-2.html)
* VMware Cloud Foundation 5.x Licenses for vCenter, ESXi, vSAN and NSX-T (VCF 5.1.1 or later supports [License Later](https://williamlam.com/2024/03/enabling-license-later-evaluation-mode-for-vmware-cloud-foundation-vcf-5-1-1.html) feature, so license keys are now optional)
* Desktop (Windows, Mac or Linux) with latest PowerShell Core and PowerCLI 12.1 Core installed. See [instructions here](https://blogs.vmware.com/PowerCLI/2018/03/installing-powercli-10-0-0-macos.html) for more details

## Management Domain Configuration

Before deploying VCF Management Domain, you will need to edit the VCF Management Domain environment configuration file, that contains all the relevant variables that are used within the deployment scripts. With the variables externalized from the deployment script, you can now have different configuration files for different environments or deployments, which are then passed to the deployment script.

See [sample-vcf-mgmt-variables.ps1](sample-vcf-mgmt-variables.ps1) for an example

This section describes the credentials to your physical vCenter Server in which the VCF lab environment will be deployed to:
```console
$VIServer = "FILL_ME_IN"
$VIUsername = "FILL_ME_IN"
$VIPassword = "FILL_ME_IN"
```

This section describes the location of the files required for deployment.
```console
$NestedESXiApplianceOVA = "/data/images/Nested_ESXi8.0u3c_Appliance_Template_v1.ova"
$CloudBuilderOVA = "/data/images/VMware-Cloud-Builder-5.2.1.1-24397777_OVF10.ova"
```

This section defines the licenses for each component within VCF. If you wish to use 60 day evaluational mode, you can leave these fields blank but you need to use VCF 5.1.1 or later
```console
$VCSALicense = ""
$ESXILicense = ""
$VSANLicense = ""
$NSXLicense = ""
```

This section defines the VCF configurations including the name of the output files for deploying the VCF Management Domain along with additional ESXi hosts to commission for use with either SDDC Manager UI or API for VCF Workload Domain deployment. The default values are sufficient.
```console
$VCFManagementDomainPoolName = "vcf-m01-rp01"
$VCFManagementDomainJSONFile = "vcf-mgmt.json"
$VCFWorkloadDomainUIJSONFile = "vcf-commission-host-ui.json"
$VCFWorkloadDomainAPIJSONFile = "vcf-commission-host-api.json"
```

This section describes the configuration for the VMware Cloud Builder virtual appliance:
```console
$CloudbuilderVMHostname = "vcf-m01-cb01"
$CloudbuilderFQDN = "vcf-m01-cb01.vcf.lab"
$CloudbuilderIP = "172.16.30.61"
$CloudbuilderAdminUsername = "admin"
$CloudbuilderAdminPassword = "VMware1!VMware1!"
$CloudbuilderRootPassword = "VMware1!VMware1!"
```

This section describes the configuration that will be used to deploy SDDC Manager within the Nested ESXi environment:
```console
$SddcManagerHostname = "vcf-m01-sddcm01"
$SddcManagerIP = "172.16.30.62"
$SddcManagerVcfPassword = "VMware1!VMware1!"
$SddcManagerRootPassword = "VMware1!VMware1!"
$SddcManagerRestPassword = "VMware1!VMware1!"
$SddcManagerLocalPassword = "VMware1!VMware1!"
```

This section defines the number of Nested ESXi VMs to deploy along with their associated IP Address(s). The names are the display name of the VMs when deployed and you should ensure these are added to your DNS infrastructure. A minimum of four hosts is required for proper VCF deployment.
```console
$NestedESXiHostnameToIPsForManagementDomain = @{
    "vcf-m01-esx01"   = "172.16.30.63"
    "vcf-m01-esx02"   = "172.16.30.64"
    "vcf-m01-esx03"   = "172.16.30.65"
    "vcf-m01-esx04"   = "172.16.30.66"
}
```

This section defines the number of Nested ESXi VMs to deploy along with their associated IP Address(s) for use in a Workload Domain deployment. The names are the display name of the VMs when deployed and you should ensure these are added to your DNS infrastructure. A minimum of four hosts should be used for Workload Domain deployment
```console
$NestedESXiHostnameToIPsForWorkloadDomain = @{
    "vcf-w01-esx01"   = "172.16.30.72"
    "vcf-w01-esx02"   = "172.16.30.73"
    "vcf-w01-esx03"   = "172.16.30.74"
    "vcf-w01-esx04"   = "172.16.30.75"
}
```

**Note:** A VCF Management Domain can be deployed with just a single Nested ESXi VM. For more details, please see this [blog post](https://williamlam.com/2023/02/vmware-cloud-foundation-with-a-single-esxi-host-for-management-domain.html) for the required tweaks.

This section describes the amount of resources to allocate to either the Nested ESXi VM(s) for use with Management Domain as well as Workload Domain (if you choose to deploy.) Depending on your usage, you may want to increase the resources but for proper functionality, this is the minimum to start with. For Memory and Disk configuration, the unit is in GB.

```console
# Nested ESXi VM Resources for Management Domain
$NestedESXiMGMTvCPU = "12"
$NestedESXiMGMTvMEM = "96" #GB
$NestedESXiMGMTCachingvDisk = "4" #GB
$NestedESXiMGMTCapacityvDisk = "500" #GB
$NestedESXiMGMTBootDisk = "32" #GB

# Nested ESXi VM Resources for Workload Domain
$NestedESXiWLDVSANESA = $false
$NestedESXiWLDvCPU = "8"
$NestedESXiWLDvMEM = "36" #GB
$NestedESXiWLDCachingvDisk = "4" #GB
$NestedESXiWLDCapacityvDisk = "250" #GB
$NestedESXiWLDBootDisk = "32" #GB
```

This section describes the Nested ESXi Networks that will be used for VCF configuration. For the ESXi management network, the CIDR definition should match the network specified in `$VMNetwork` variable.
```console
$NestedESXiManagementNetworkCidr = "172.16.0.0/16" # should match $VMNetwork configuration
$NestedESXivMotionNetworkCidr = "172.30.32.0/24"
$NestedESXivSANNetworkCidr = "172.30.33.0/24"
$NestedESXiNSXTepNetworkCidr = "172.30.34.0/24"
```

This section describes the configurations that will be used to deploy the VCSA within the Nested ESXi environment:
```console
$VCSAName = "vcf-m01-vc01"
$VCSAIP = "172.16.30.67"
$VCSARootPassword = "VMware1!"
$VCSASSOPassword = "VMware1!"
$EnableVCLM = $true
```

This section describes the configurations that will be used to deploy the NSX-T infrastructure within the Nested ESXi environment:
```console
$NSXManagerSize = "medium"
$NSXManagerVIPHostname = "vcf-m01-nsx01"
$NSXManagerVIPIP = "172.16.30.68"
$NSXManagerNode1Hostname = "vcf-m01-nsx01a"
$NSXManagerNode1IP = "172.16.30.69"
$NSXRootPassword = "VMware1!VMware1!"
$NSXAdminPassword = "VMware1!VMware1!"
$NSXAuditPassword = "VMware1!VMware1!"
```

This section describes the location as well as the generic networking settings applied to Nested ESXi & Cloud Builder VMs:

```console
$VMDatacenter = "Datacenter"
$VMCluster = "Cluster"
$VMNetwork = "Workloads"
$VMDatastore = "vsanDatastore"
$VMNetmask = "255.255.0.0"
$VMGateway = "172.16.1.53"
$VMDNS = "172.16.1.3"
$VMNTP = "172.16.1.53"
$VMPassword = "VMware1!"
$VMDomain = "vcf.lab"
$VMSyslog = "172.16.30.100"
$VMFolder = "wlam-vcf52"
```

> **Note:** It is recommended that you use an NTP server that has both forward and DNS resolution configured. If this is not done, during the VCF JSON pre-req validation phase, it can take longer than expected for the DNS timeout to complete prior to allowing user to continue to VCF deployment.

### Workload Domain Configuration

Before deploying a VCF Workload Domain, you will need to edit the Workload Domain environment configuration file, that contains all the relevant variables that are used within the deployment scripts. With the variables externalize from the deployment script, you can now have different configuration files for different environments or deployments, which are then passed to the deployment script.

See [sample-vcf-wld-variables.ps1](sample-vcf-wld-variables.ps1) for an example


This section describes the credentials to your deployed SDDC Manager from setting up the Management Domain:
```console
$sddcManagerFQDN = "FILL_ME_IN"
$sddcManagerUsername = "administrator@vsphere.local"
$sddcManagerPassword = "VMware1!"
```

This section defines the licenses for each component within VCF
```console
$ESXILicense = "FILL_ME_IN"
$VSANLicense = "FILL_ME_IN"
$NSXLicense = "FILL_ME_IN"
```

This section defines the Management and Workload Domain configurations, which the default values should be sufficient unless you have modified anything from the original deployment script
```console
$VCFManagementDomainPoolName = "vcf-m01-rp01"
$VCFWorkloadDomainAPIJSONFile = "vcf-commission-host-api.json"
$VCFWorkloadDomainName = "wld-w01"
$VCFWorkloadDomainOrgName = "vcf-w01"
$EnableVCLM = $true
$VLCMImageName = "Management-Domain-ESXi-Personality" # Ensure this label matches in SDDC Manager->Lifecycle Management->Image Management
$EnableVSANESA = $false
```

> **Note:** If you're going to deploy VCF Workload Domain with vLCM enabled, make sure the `$VLCMImageName` name matches what you see in SDDC Manager under Lifecycle Management->Image Management. In VCF 5.2, the default name should be "Management-Domain-ESXi-Personality" and in VCF 5.1.x the default name should be "Management-Domain-Personality" but best to confirm before proceeding with deployment.

This section defines the vCenter Server configuration that will be used in the Workload Domain
```console
$VCSAHostname = "vcf-w01-vc01"
$VCSAIP = "172.16.30.76"
$VCSARootPassword = "VMware1!VMware1!"
```

This section defines the NSX Manager configurations that will be used in the Workload Domain
```console
$NSXManagerVIPHostname = "vcf-w01-nsx01"
$NSXManagerVIPIP = "172.16.30.77"
$NSXManagerNode1Hostname = "vcf-w01-nsx01a"
$NSXManagerNode1IP = "172.16.30.78"
$NSXManagerNode2Hostname = "vcf-w01-nsx01b"
$NSXManagerNode2IP = "172.16.30.79"
$NSXManagerNode3Hostname = "vcf-w01-nsx01c"
$NSXManagerNode3IP = "172.16.30.80"
$NSXAdminPassword = "VMware1!VMware1!"
$SeparateNSXSwitch = $false
```

> **Note:** See [VMware Cloud Foundation with a single ESXi host for Workload Domain?](https://williamlam.com/2023/02/vmware-cloud-foundation-with-a-single-esxi-host-for-workload-domain.html) if you only want to deploy 1 NSX Manager.

This section defines basic networking information that will be needed to deploy vCenter and NSX components
```console
$VMNetmask = "255.255.0.0"
$VMGateway = "172.16.1.53"
$VMDomain = "vcf.lcm"
```

## Logging

There is additional verbose logging that outputs as a log file in your current working directory **vcf-lab-deployment.log**

## Sample Execution

In the example below, I will be using a one /16 VLANs (172.16.0.0/16) with the following allocation in DNS

|           Hostname          | IP Address    | Function             |
|:---------------------------:|---------------|----------------------|
| vcf-m01-cb01.vcf.lab        | 172.16.30.61 | Cloud Builder         |
| vcf-m01-sddcm01.vcf.lab     | 172.16.30.62 | SDDC Manager          |
| vcf-m01-vc01.vcf.lab        | 172.16.30.67 | vCenter Server        |
| vcf-m01-nsx01.vcf.lab       | 172.16.30.68 | NSX-T VIP             |
| vcf-m01-nsx01a.vcf.lab      | 172.16.30.69 | NSX-T Node 1          |
| vcf-m01-esx01.vcf.lab       | 172.16.30.63 | ESXi Host 1 for Mgmt  |
| vcf-m01-esx02.vcf.lab       | 172.16.30.64 | ESXi Host 2 for Mgmt  |
| vcf-m01-esx03.vcf.lab       | 172.16.30.65 | ESXi Host 3 for Mgmt  |
| vcf-m01-esx04.vcf.lab       | 172.16.30.66 | ESXi Host 4 for Mgmt  |
| vcf-w01-esx01.vcf.lab       | 172.16.30.72 | ESXi Host 5 for WLD   |
| vcf-w01-esx02.vcf.lab       | 172.16.30.73 | ESXi Host 6 for WLD   |
| vcf-w01-esx03.vcf.lab       | 172.16.30.74 | ESXi Host 7 for WLD   |
| vcf-w01-esx04.vcf.lab       | 172.16.30.75 | ESXi Host 8 for WLD   |

### Deploy Nested ESXi and Cloud Builder VMs

Here is a screenshot of running the script if all basic pre-reqs have been met and the confirmation message before starting the deployment:

![](screenshots/screenshot-1.png)

Here is an example output of a complete deployment:

![](screenshots/screenshot-2.png)

**Note:** Deployment time will vary based on underlying physical infrastructure resources. In my lab, this took ~19min to complete.

Once completed, you will end up with eight Nested ESXi VM and VMware Cloud Builder VMs which is placed into a vApp.

![](screenshots/screenshot-3.png)

### Deploy VCF Management Domain

By default, the script will auto generate the required VCF Management Domain deployment file `vcf-mgmt.json` based off of your specific deployment and save that into the current working directory. Additionally, the VCF deployment file will automatically be submitted to SDDC Manager and begin the VCF Bringup process, which in previous versions of this script was performed manually by the end user.

Now you can just open a web browser to your SDDC Manager deployment and monitor the VCF Bringup progress.

![](screenshots/screenshot-4.png)

**Note:** If you wish to disable the VCF Bringup process, simply search for the variable named `$startVCFBringup` in the script and change the value to 0.

The deployment and configuration can take up to several hours to complete depending on the resources of your underlying hardware. In this example, the deployment took about ~1.5 to complete and you should see a success message as shown below.

![](screenshots/screenshot-6.png)

Click on the Finish button which should prompt you to login to SDDC Manager. You will need to use `administrator@vsphere.local` credentials that you had configured within the deployment script for the deployed vCenter Server.

![](screenshots/screenshot-7.png)

### Deploy VCF Workload Domain

## Manual Method

By default, the script will auto generate the VCF Workload Domain host commission file `vcf-commission-host-ui.json` based off of your specific deployment and save that into the current working directory.

Once the VCF Management Domain has been deployed, you can login to SDDC Manager UI and under `Inventory->Hosts`, click on the `COMMISSION HOSTS` button and upload the generated JSON configuration file.

**Note:** There is currently a different JSON schema between the SDDC Manager UI and API for host commission and the generated JSON file can only be used by SDDC Manager UI. For the API, you need to make some changes to the file including replacing the networkPoolName with the correct networkPoolId. For more details, please refer to the JSON format in the [VCF Host Commission API]([text](https://developer.broadcom.com/xapis/vmware-cloud-foundation-api/latest/v1/hosts/post/))

![](screenshots/screenshot-8.png)

Once the ESXi hosts have been added to SDDC Manager, then you can perform a manual VCF Workload Domain deployment using either the SDDC Manager UI or API.

![](screenshots/screenshot-9.png)

## Automated Method

A supplemental automation script [vcf-automated-workload-domain-deployment.ps1](vcf-automated-workload-domain-deployment.ps1) will be used to automatically standup the workload domain. It will assume that the VCF Workload Domain host commission file `vcf-commission-host-api.json` was generated from running the initial deployment script and this file will contain a "TBD" field because the SDDC Manager API expects the Management Domain Network Pool ID, which will be retrieved automatically as part of using the additional automation.

Here is an example of what will be deployed as part of Workload Domain creation:

|           Hostname          | IP Address    | Function       |
|:---------------------------:|---------------|----------------|
| vcf-w01-vc01.vcf.lab    | 172.16.30.76 | vCenter Server |
| vcf-w01-nsx01.vcf.lab   | 172.16.30.77 | NSX-T VIP      |
| vcf-w01-nsx01a.vcf.lab  | 172.16.30.78 | NSX-T Node 1   |
| vcf-w01-nsx01b.vcf.lab  | 172.16.30.79 | NSX-T Node 2   |
| vcf-w01-nsx01c.vcf.lab  | 172.16.30.80 | NSX-T Node 3   |

> **Note:** See [VMware Cloud Foundation with a single ESXi host for Workload Domain?](https://williamlam.com/2023/02/vmware-cloud-foundation-with-a-single-esxi-host-for-workload-domain.html) if you only want to deploy 1 NSX Manager.

### Example Deployment

Here is a screenshot of running the script if all basic pre-reqs have been met and the confirmation message before starting the deployment:

![](screenshots/screenshot-10.png)

Here is an example output of a completed deployment:

![](screenshots/screenshot-11.png)

**Note:** While the script should finish in ~3-4 minutes, the actual creation of the Workload Domain will take a bit longer and will depend on your resources.

To monitor the progress of your Workload Domain deployment, login to the SDDC Manager UI:

![](screenshots/screenshot-12.png)

![](screenshots/screenshot-13.png)

If you now login to your vSphere UI for your Management Domain, you should see the following inventory:

![](screenshots/screenshot-14.png)
