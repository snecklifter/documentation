---
title: How to install Red Hat Update Infrastructure on an existing VMware virtual machine
description: Shows you how to update your existing hosts to target UKCloud's approved Red Hat Update Infrastructure (RHUI)
services: vmware
author: Sue Highmoor
reviewer: Paul Cantle
lastreviewed: 04/07/2020
toc_rootlink: How To
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Install Red Hat Update Infrastructure on an existing virtual machine
toc_fullpath: How To/vmw-how-install-rhui-vm.md
toc_mdlink: vmw-how-install-rhui-vm.md
---

# How to install Red Hat Update Infrastructure on an existing virtual machine

## Overview

This article provides advice on how to update your existing Red Hat virtual machines (VMs) to target UKCloud's approved Red Hat Update Infrastructure (RHUI).

As of July 2015, UKCloud implemented RHUI to provide automatic updates to our Red Hat customers on our Assured OFFICIAL and Elevated OFFICIAL security domains. This provides benefits such as the reliable availability of Red Hat approved patches and supported repositories.

This updated service replaces the previous UKCloud RHUI.

## Changes from the previous RHUI

There are several differences between the original UKCloud RHUI and the latest one documented here. One of the major changes is there is now no longer a requirement to differentiate the entitlement RPMs (that grant access to the repositories) between Assured and Elevated. Both have a common hostname. This means a single file (per Operating System version) can be maintained across both environments.

## Prerequisites

There are two pre-requisites required for using UKCloud's RHUI:

- The host can resolve the RHUI CDS DNS records (contact Customer Support for the IP addresses for the Elevated records). For Elevated, you can achieve this by configuring an A record in your local DNS, or configuring an `/etc/hosts` file to point to rh-cds.ukcloud.com. For Assured/Internet, you can use the Internet DNS name of rh-cds.ukcloud.com without any additional DNS configuration.

- All hosts using the service must be able to access the RHUI Content Delivery Server (CDS) on port 443 (HTTPS). Ensure that you have configured SNAT and firewall policies on your edge gateway device. If you have any questions on this, contact UKCloud Support.

## Installation

You can browse the all entitlement RPM files in the following location:

- [ALL RPMS](https://rh-cds.ukcloud.com/redhat/client_rpms/)

Or specific RPMs for direct download are located as follows:

- [RHEL 6 Standard](https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL6-STANDARD-2.0-2.noarch.rpm)

- [RHEL 6 Extended Update Support](https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL6-EUS-2.0-1.noarch.rpm)

- [RHEL 7 Standard](https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL7-STANDARD-2.0-2.noarch.rpm)

- [RHEL 7 Extended Update Support](https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL7-EUS-2.0-1.noarch.rpm)

- [RHEL 8 Standard](https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL8-STANDARD-2.0-1.noarch.rpm)

If you require the high availability (HA) package, raise a Service Request directly via the [My Calls](https://portal.skyscapecloud.com/support/ivanti) section of the UKCloud Portal and we'll provide you with a download location.

1. Download the appropriate file to your VM (or place the contents in an accessible location such as NFS share or FTP).

2. Install the relevant RPM for your Operating System.

   If your system has an entitlement RPM from UKCloud's previous RHUI, then that will need to be removed prior to installing the latest entitlement RPM
   
   For example, for RHEL6 or RHEL7 :
   ```bash
   yum erase $(rpm -qa|grep IL2-Client)
   ```
   
   Install the new entitlement RPM
   
   For example, for RHEL6 :
   ```bash
   cd /tmp
   wget --no-check-certificate https://rh-cds.ukcloud.com/redhat/client_rpms/UKCloud-Client-RHEL6-STANDARD-2.0-2.noarch.rpm
   yum install UKCloud-Client-RHEL6-STANDARD-2.0-2.noarch.rpm
   ```
  
3. Clean yum and clear the cache:
   ```bash 
   yum clean all; rm -rf /var/cache/yum
   ```

4. Test the RHUI is working:
   ```bash
   yum update
   ```

5. The first time you update from RHUI you may be prompted to accept the following two certificates:

    - The Red Hat entitlement certificate

    - The Client entitlement certificate
    
## Upgrading the RPMs to a newer version

On occassion, it may be required to update the entitlement RPM local to your systems. This will usually be due to additional repositories being requested and added into the entitlement. If you don't require access to the newly added repositories, then you do not need to install the later version of the RPM

## Troubleshooting

The primary issues you may encounter are:

### DNS Failure

Check the DNS lookup is working and you have the correct entry for Assured and Elevated.

### 443 not accessible

1. Check your firewall configuration, including local firewalls (`iptables`) and edge gateway.

2. Ensure you have the correct destination IP entry for Assured or Elevated.

### Incorrect version

Ensure you have installed the correct RPM for your Operating System release.

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
