---
title: Fixing your IPsec VPN following upgrade to NSX 6.4
description: Describes how to resolve issues with IPsec VPN after upgrading to NSX 6.4
services: vmware
author: Sue Highmoor
reviewer: lthangarajah
lastreviewed: 20/08/2019
toc_rootlink: Reference
toc_sub1: 
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Fixing your IPsec VPN following upgrade to NSX 6.4
toc_fullpath: Reference/vmw-ref-trouble-ipsec-nsx64.md
toc_mdlink: vmw-ref-trouble-ipsec-nsx64.md
---

# Fixing your IPsec VPN following upgrade to NSX 6.4

With the upgrade to NSX version 6.4, VMware have made a security change to the default settings of the NSX edge gateway's IPsec VPN service. The new default for IPsec VPN service is to use the DH-14 (Diffie–Hellman group 14) algorithm during phase 1 negotiation of the VPN tunnel. You have the option of selecting a different version of Diffie-Hellman in the advanced gateway GUI. The following versions are supported:

- DH-2 (Diffie–Hellman group 2)

- DH-5 (Diffie–Hellman group 5)

- DH-14 (Diffie–Hellman group 14)

- DH-15 (Diffie–Hellman group 15)

- DH-16 (Diffie–Hellman group 16)

However, during the upgrade of one of our regions we discovered that for edges that have not been converted to advanced gateways that have VPNs configured, the upgrade changes the config on the VPN tunnels to use the DH-14 algorithm instead of the previously only available DH-2.

As a result, VPN tunnels are then broken until either of the following is performed:

- The edge with the affected VPN is upgraded to an advanced gateway and the VPN config is changed from DH-14 back to DH-2

- The peer end config is amended to use DH-14 for its ike negotiation

UKCloud support staff can change the VPN config back to DH-2 on a temporary basis until the edge is converted, however the setting is not persistent, and under certain configuration changes to an edge, the setting of DH-14 may be reapplied. Converting to an advanced gateway does not require any downtime nor cause any outage during the conversion.

Follow the steps below to convert your edge to an advanced gateway and configure your VPN back to DH-2. To complete these steps, you must have access to the UKCloud Portal and vCloud Director.

## Change your VPN settings

To change the DH setting on your broken VPN:

1. In the vCloud Director *Virtual Datacenters* dashboard, select the VDC that contains the edge with the broken VPN.

2. In the left navigation panel, click **Edges**.

    ![Edges menu option in vCloud Director](images/vmw-vcd-mnu-edges.png)

3. Select the edge gateway and click **Configure Services**.

    ![Configure Services button](images/vmw-vcd-edge-btn-config.png)

4. Select the **VPN** tab.

5. On the *IPsec VPN Configuration* page, select the **IPsec VPN Sites** tab.

6. Select the VPN tunnel that is broken and click the edit icon.

7. From the **Diffee-Hellman Group** list, select DH2.

    ![Diffie-Helman Group list](images/vmw-tp-ipsec-fix2.png)

    You may need to scroll the window to the bottom of the page to find the list.

8. When you're done, click **Keep**.

9. Repeat the steps 6 and 7 for any other VPNs that are broken and when you're ready to save the configuration, click **Save Changes** at the top of the page.

    Check the state of the VPN by clicking the **Statistics** tab at the top of the screen and selecting **IPSEC VPN** to view the health of your VPN.

## Feedback

If you find a problem with this article, click **Improve this Doc** to make the change yourself or raise an [issue](https://github.com/UKCloud/documentation/issues) in GitHub. If you have an idea for how we could improve any of our services, send an email to <feedback@ukcloud.com>.
