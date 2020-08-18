---
title: Fixing your advanced gateway if you encounter NullPointerException error
description: Intended for customers who are experiencing the NullPointerException error when attempting to view their edge gateway properties
services: vmware
author: Sue Highmoor
reviewer: lthangarajah
lastreviewed: 20/08/2019
toc_rootlink: Reference
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Fixing your advanced gateway if you encounter NullPointerException error
toc_fullpath: Reference/vmw-ref-trouble-null-pointer.md
toc_mdlink: vmw-ref-trouble-null-pointer.md
---

# Fixing your advanced gateway if you encounter NullPointerException error

With the upgrade to NSX version 6.4, edges that have been converted to an advanced gateway may produce an error when attempting to view the properties of the edge gateway in vCloud Director. This does not affect access to the tenant portal GUI, where you can set up firewall rules and so on, this only affects viewing the edge properties such as the IP addresses assigned to an edge.

To complete the steps in this article, you must have access to vCloud Director.

To fix your edge to enable viewing of its properties:

1. In the vCloud Director *Virtual Datacenters* dashboard, select the VDC that contains the edge with the broken VPN.

2. In the left navigation panel, click **Edges**.

    ![Edges menu option in vCloud Director](images/vmw-vcd-mnu-edges.png)

3. Select the edge gateway and click **Configure Services**.

    ![Configure Services button](images/vmw-vcd-edge-btn-config.png)

4. Select the **Routing** tab.

5. Select **Routing Configuration**.

6. Under the *Static Routing Default Gateway* section, The **MTU** field will contain a value of `0`. Change the value from `0` to `1500`.

7. When you're done, click **Save Changes** at the top of the page.

8. Close vCloud Director and attempt to view the properties of the edge gateway in vCloud.

    You should no longer get the `NullPointerException` error and the properties should be displayed.

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
