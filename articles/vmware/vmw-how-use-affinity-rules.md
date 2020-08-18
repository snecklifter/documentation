---
title: How to use affinity and anti-affinity rules
description: Describes how to use affinity and anti-affinity rules to control the movement of VMs around the UKCloud platform
services: vmware
author: Sue Highmoor
reviewer: lthangarajah
lastreviewed: 21/06/2019
toc_rootlink: How To
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Use affinity and anti-affinity rules
toc_fullpath: How To/vmw-how-use-affinity-rules.md
toc_mdlink: vmw-how-use-affinity-rules.md
---

# How to use affinity and anti‑affinity rules

## Overview

Affinity and anti-affinity rules determine whether or not VMs are kept together as they are moved around within your environment.

- Affinity rules keep VMs together on the same host

- Anti-affinity rules ensure that VMs are distributed across different hosts

For example, if it is important that your application has very low latency, you can create affinity rules to make sure that the application's VMs are always kept on the same host. If you want your application to be robust in the event of host failures, you can create anti-affinity rules to spread the VMs across multiple hosts.

> [!IMPORTANT]
> Excessive use of affinity and anti-affinity rules can cause unpredictable VM and platform performance issues. For example, VMs involved in rules may have an increased likelihood of migrating between hosts simultaneously. Adding a large number of VMs to an affinity rule may cause performance issues if all of the VMs become busy simultaneously. An anti-affinity rule with a large number of VMs may not be honoured if the platform is unable to place the VMs on separate hosts.
>
> UKCloud reserves the right to disable or disregard rules if required for maintenance or platform management.

### Intended audience

To complete the steps in this guide you must have access to the UKCloud Portal and vCloud Director.

## Creating an affinity or anti-affinity rule

To create an affinity or anti-affinity rule:

1. In vCloud Director, click the VDC in which you want to create the affinity or anti-affinity rule.

2. In the left navigation panel, select **Affinity Rules**.

    ![Affinity rules option](images/vmw-vcd-mnu-affinity.png)

3. To create an affinity rule, in the *Affinity Rules* section, click **New**.

    To create an anti-affinity rule,in the *Anti-Affinity Rules* section, click **New**.

    ![New Affinity rule links](images/vmw-vcd-new-affinity-rule.png)

4. In the *Create Affinity Rule* or *Create Anti-Affinity Rule* dialog box, enter a **Name** for the rule.

5. If you don't want the rule to take effect immediately, deselect the **Enabled** option.

    If you leave this option selected, the rule is applied and the selected VMs are redistributed as required as soon as you create the rule.

6. If you want the VMs added to the rule to be powered on if the rule cannot be met, deselect the **Required** option.

    If you leave this option selected, if the rule cannot be met, the VMs added to the rule will not power on.

7. In the *VMs in Rule* section, select the VMs that you want to add to the rule.

    ![Create Affinity Rule dialog box](images/vmw-vcd-create-affinity-rule.png)

    > [!TIP]
    > Select the **Show selected** option to see a list of the VMs already included in the rule.

8. When you're done, click **Save**.

## Next steps

For more information about UKCloud for VMware see the [*Getting Started Guide for UKCloud for VMware*](vmw-gs.md)

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
