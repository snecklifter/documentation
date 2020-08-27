---
title: How to deploy a Kubernetes cluster on Azure Stack Hub using AKS engine
description: Deploy a Kubernetes cluster using AKS engine
services: azure-stack
author: Daniel Brennand
reviewer: Daniel Brennand
lastreviewed:

toc_rootlink: Users
toc_sub1: How To
toc_sub2:
toc_sub3:
toc_sub4:
toc_title: Deploy a Kubernetes cluster using AKS engine
toc_fullpath: Users/How To/azs-how-create-aksengine.md
toc_mdlink: azs-how-create-aksengine.md
---

# How to deploy a Kubernetes cluster on Azure Stack Hub using AKS engine

## Overview

The [AKS engine command-line utility](https://github.com/Azure/aks-engine/tree/master/) allows Azure Stack Hub tenants to deploy and manage a Kubernetes cluster.
AKS engine can be used to create, upgrade, scale and maintain Azure Resource Manager native clusters running on VMs and other infrastructure-as-a-service (IaaS) resources on Azure Stack Hub.

[Kubernetes on Azure Stack Hub in GA](https://azure.microsoft.com/en-gb/updates/kubernetes-on-azure-stack-in-ga/)

## Useful links

This article is based on Microsoft's deploy documentation for Kubernetes on [Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2005).

* [What is the AKS engine on Azure Stack Hub?](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-2005)

* [Set up the prerequisites for the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-set-up?view=azs-2005)

* [Install the AKS engine on Linux in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2005)

* [Install the AKS engine on Windows in Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2005)

* [Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster?view=azs-2005)

> [!CAUTION]
> Before proceeding, ensure you are aware of the [issues and limitations](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) of Kubernetes on Azure Stack Hub.

### Intended audience

To complete the steps in this article, you must have appropriate access to a subscription in Azure Stack Hub.

## Prerequisites

### Operator prerequisites

Ask your Azure Stack Hub operator if the following prerequisites below have been met:

* Azure Stack Hub version >= **2005**

* Linux custom script extension version >= **2.0.6**

* AKS base Ubuntu image >= **2020.05.13**

    - **Image name:** AKS Base Ubuntu 16.04-LTS Image Distro

    - **Offer:** aks

    - **Publisher:** microsoft-aks

    - **SKU:** aks-ubuntu-1604-202005

### Tenant prerequisites

* Have appropriate access to a subscription in Azure Stack Hub.

* Ability to create a Service principal name (SPN) with **contributor** role assigned to it.

    > [!NOTE]
    > This is used by AKS engine to deploy the Kubernetes cluster on Azure Stack Hub.

* PowerShell 5.1

* [Git CLI](https://git-scm.com/)

* OpenSSH

    > [!NOTE]
    > This is usually located at `C:\Windows\System32\OpenSSH`

    Be able to use the following commands from OpenSSH:

    - `ssh-agent`

    - `ssh-keygen`

* [`AzureStack` PowerShell module](https://docs.ukcloud.com/articles/azure/azs-how-configure-powershell-users.html#install-azure-stack-hub-powershell)

* `AzureRM.OperationalInsights` PowerShell module:

    ```powershell
    Install-Module -Name AzureRM.OperationalInsights -RequiredVersion 5.0.6 -Force -Verbose
    ```

* `AzureAD` PowerShell module:

    ```powershell
    Install-Module -Name AzureAD -Force -Verbose
    ```

* `Posh-SSH >= 2.3.0` PowerShell module:

    ```powershell
    Install-Module -Name Posh-SSH -MinimumVersion 2.3.0 -Force -Verbose -AllowPreRelease
    ```

## Deployment

UKCloud has created a script ([DeployAshKubernetes.ps1](linkhere)) and module ([AksEngine.psm1](linkhere)) to automate the deployment process of a Kubernetes cluster on Azure Stack Hub.

The [DeployAshKubernetes.ps1](linkhere) creates the following high level resources:

1. The AKS engine client VM (either Linux or Windows), which is used to deploy the Kubernetes cluster on Azure Stack Hub.

2. A Kubernetes cluster deployed at the [latest version supported on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) (Kubernetes version: 1.16.9)

### AKS engine client VM

The AKS engine client VM can be deployed as either Linux (Ubuntu 18.04 LTS) or Windows (Server 2019 with containers).

The client VM comes with **Kubectl** and **Helm** configured out of the box ready for you to interact with the Kubernetes cluster.

For Linux, **SSH server** is used to interact with the client VM and run automation tasks against it. Whereas, for Windows; **WinRM** is used.
For additional security, the following Network Security Group (NSG) rules are configured:

* Linux client VM: **SSH** access (port 22) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`

* Windows client VM: **WinRM** access (ports 5985 and 5986) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`

    > [!IMPORTANT]
    > For **WinRM** to connect successfully, **TrustedHosts** is configured on the local machine (with the Windows client VM's IP added) and on the Windows client VM (with the value(s) from `$AllowedIpAddresses`).
    >
    > For **WinRM** to be configured successfully, you **must** have appropriate permissions to edit **TrustedHosts** on your local machine. If you are using **Group Policy**; this may not work.
    >
    > For more information, see [WinRM configuration and remote management](https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management).

* Kubernetes cluster: **SSH** access (port 22) is only permitted from the `$AllowedIpAddresses` variable or defaults to the returned value of `Invoke-RestMethod -UseBasicParsing -Uri "ifconfig.me" -Method GET`. Furthermore, the AKS engine client VM's IP is also permitted.

> [!NOTE]
> For the Windows client VM deployment, we spent a lot of time troubleshooting issues with OpenSSH server.
> We experienced the following issues:
>
> * We could not achieve SSH agent forwarding from the client VM -> Kubernetes master node ->  Kubernetes worker node
>
> * We managed to achieve the first hop (from the client VM to the Kubernetes master node) however, we couldn't achieve the final hop to the Kubernetes worker node(s).
>
> * We made sure that each ssh-agent configuration used `ForwardAgent yes` and the SSH server configuration with `AllowAgentForwarding yes` set.
>
> * We tested with PowerShell 5 and 7.
>
> * We experienced that with an active RDP session, everything began working as expected.

### Enabling cluster monitoring

You can choose to enable monitoring of your Kubernetes cluster on Azure Stack Hub. There are two methods provided by Microsoft.

> [!NOTE]
> In order to enable monitoring for your Kubernetes cluster on Azure Stack Hub, you will need an active public Azure subscription.

#### Prerequisite step

Both methods require the prerequisite of using an [Azure Resource Manager (ARM) template](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) to enable the **ContainerInsights** solution for a **Log Analytics Workspace**.

#### [Method One](https://docs.microsoft.com/en-us/azure-stack/user/kubernetes-aks-engine-azure-monitor?view=azs-2005#method-one)

> [!IMPORTANT]
> This method is **not** used by the [DeployAshKubernetes.ps1](linkhere) script.
> This method should be used if you decide to enable monitoring for an already deployed Kubernetes cluster on Azure Stack Hub which doesn't have monitoring enabled.

1. Use the following PowerShell script to enable the **ContainerInsights** solution for a **Log Analytics Workspace**:

    Enter details below to provide values for the variables in the following script in this article:

    | Variable name   | Variable description                                               | Input            |
    |-----------------|--------------------------------------------------------------------|------------------|
    | \$LogAnalyticsWorkspaceName    | The name of the Log Analytics Workspace in public Azure.                 | <form oninput="result.value=loganalyticsworkspacename.value" id="loganalyticsworkspacename" style="display: inline;"><input type="text" id="loganalyticsworkspacename" name="loganalyticsworkspacename" style="display: inline;" placeholder="kubernetes-cluster-workspace"/></form> |
    | \$LogAnalyticsWorkspaceResourceGroupName    | The name of the resource group containing the Log Analytics Workspace in public Azure.                 | <form oninput="result.value=loganalyticsworkspaceresourcegroupname.value" id="loganalyticsworkspaceresourcegroupname" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupname" name="loganalyticsworkspaceresourcegroupname" style="display: inline;" placeholder="kubernetes-cluster-workspace-rg"/></form> |
    | \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics Workspace.                 | <form oninput="result.value=loganalyticsworkspacelocation.value" id="loganalyticsworkspacelocation" style="display: inline;"><input type="text" id="loganalyticsworkspacelocation" name="loganalyticsworkspacelocation" style="display: inline;" placeholder="UK South"/></form> |
    | \$DownloadDirectoryPath    | The local path used to download the ARM template files to                  | <form oninput="result.value=downloaddirectorypath.value" id="downloaddirectorypath" style="display: inline;"><input type="text" id="downloaddirectorypath" name="downloaddirectorypath" style="display: inline;" placeholder="C:\Temp\EnableMonitoring"/></form> |

    <pre><code class="language-PowerShell"># Declare variables
    $LogAnalyticsWorkspaceName = "<output form="loganalyticsworkspacename" name="result" style="display: inline;">kubernetes-cluster-workspace</output>"
    $LogAnalyticsWorkspaceResourceGroupName = "<output form="loganalyticsworkspaceresourcegroupname" name="result" style="display: inline;">kubernetes-cluster-workspace-RG</output>"
    $LogAnalyticsWorkspaceLocation = "<output form="loganalyticsworkspacelocation" name="result" style="display: inline;">UK South</output>"
    $DownloadDirectoryPath = "<output form="downloaddirectorypath" name="result" style="display: inline;">C:\Temp\EnableMonitoring</output>"
    $MonitoringARMTemplateUri = "https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature_prod/docs/templates/azuremonitor-containerSolution.json"
    $MonitoringARMTemplateParamsUri = "https://raw.githubusercontent.com/microsoft/OMS-docker/ci_feature_prod/docs/templates/azuremonitor-containerSolutionParams.json"

    # Connect to public Azure
    Connect-AzureRmAccount

    # Check if the Log Analytics resource group exists, If it doesn't exist then create it
    try {
        $LogAnalyticsWorkspaceResourceGroup = Get-AzureRmResourceGroup -Name $LogAnalyticsWorkspaceResourceGroupName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
    }
    catch {
        Write-Warning -Message "Log Analytics workspace resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"" doesn't exist. Creating now..."
        try {
            $LogAnalyticsWorkspaceResourceGroup = New-AzureRmResourceGroup -Name $LogAnalyticsWorkspaceResourceGroupName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
        }
        catch {
            throw "Failed to create Log Analytics workspace resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"":`n$($_.Exception.Message)"
        }
    }
    # Check if the Log Analytics workspace exists, If it doesn't exist then create it
    try {
        Write-Verbose "Attempting to retrieve Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" in resource group ""$($LogAnalyticsWorkspaceResourceGroupName)""."
        $LogAnalyticsWorkspace = Get-AzureRmOperationalInsightsWorkspace -ResourceGroupName $LogAnalyticsWorkspaceResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspaceName -ErrorAction Stop
    }
    catch {
        try {
            Write-Warning -Message "Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" doesn't exist. Creating now..."
            $LogAnalyticsWorkspace = New-AzureRmOperationalInsightsWorkspace -ResourceGroupName $LogAnalyticsWorkspaceResourceGroup.ResourceGroupName -Name $LogAnalyticsWorkspaceName -Location $LogAnalyticsWorkspaceLocation -ErrorAction Stop
        }
        catch {
            throw "Failed to create Log Analytics workspace ""$($LogAnalyticsWorkspaceName)"" in resource group ""$($LogAnalyticsWorkspaceResourceGroupName)"":`n$($_.Exception.Message)"
        }
    }

    # Download required ARM template to enable the ContainerInsights solution
    # Ensure C:\Temp\EnableMonitoring directory exists
    New-Item -ItemType "Directory" -Path $DownloadDirectoryPath
    # Download the required ARM template files
    Invoke-WebRequest -Uri $MonitoringARMTemplateUri -OutFile "$($DownloadDirectoryPath)\containerSolution.json"
    Invoke-WebRequest -Uri $MonitoringARMTemplateParamsUri -OutFile "$($DownloadDirectoryPath)\containerSolutionParams.json"
    # Obtain current subscription ID from public Azure
    $SubscriptionID = (Get-AzureRmSubscription).SubscriptionId
    # Set the public Azure subscription
    Set-AzureRmContext -SubscriptionId $SubscriptionID
    # Edit the ARM template parameters file
    $MonitoringARMTemplateParamsJson = Get-Content -Path "$($DownloadDirectoryPath)\containerSolutionParams.json" | ConvertFrom-Json
    $MonitoringARMTemplateParamsJson.parameters.workspaceResourceId.value = $LogAnalyticsWorkspace.ResourceId
    $MonitoringARMTemplateParamsJson.parameters.workspaceRegion.value = $LogAnalyticsWorkspace.Location
    # Set the content of the ARM template parameters file
    $MonitoringARMTemplateParamsJson | ConvertTo-Json -Depth 20 | Set-Content -Path "$($DownloadDirectoryPath)\containerSolutionParams.json"
    # Deploy the ARM template to enable ContainerInsights solution
    New-AzureRmResourceGroupDeployment -Name "OnboardCluster" -ResourceGroupName $LogAnalyticsWorkspaceResourceGroupName -TemplateFile "$($DownloadDirectoryPath)\containerSolution.json" -TemplateParameterFile "$($DownloadDirectoryPath)\containerSolutionParams.json"

    # Output the Log Analytics workspace ID and workspace key
    Write-Output -InputObject "Please make a note of these values, they are required later."
    Write-Output -InputObject "Log Analytics workspace ID ""$($LogAnalyticsWorkspace.CustomerId)""."
    $WorkspaceKey = (Get-AzureRmOperationalInsightsWorkspaceSharedKeys -ResourceGroupName $LogAnalyticsWorkspace.ResourceGroupName -Name $LogAnalyticsWorkspace.Name).PrimarySharedKey
    Write-Output -InputObject "Log Analytics workspace key ""$($WorkspaceKey)""."
    </code></pre>

2. SSH or create a WinRM session to the AKS engine client VM.

3. Add the required Helm repository: `helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/`

4. Run the following command, replacing the **workspace_id**, **workspace_key** (These were outputted in the script above) and **cluster_name** variables to install the Helm chart on the Kubernetes cluster:

    ```bash
    helm install --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<workspace_id>,omsagent.secret.key=<workspace_key>,omsagent.env.clusterName=<cluster_name> myrelease-1 incubator/azuremonitor-containers
    ```

    > [!NOTE]
    > This Helm chart deploys the required OMS agent containers into the **kube-system** namespace in the Kubernetes cluster.

    > [!TIP]
    > You can find the **cluster_name** by running the following command on the Kubernetes master node:
    > ```bash
    > azureuser@k8s-master-85734688-0:~$ kubectl cluster-info
    > Kubernetes master is running at https://kubemaster89296451.frn00006.cloudapp.azure.ukcloud.com
    > ```
    >
    > The **cluster_name** in this example is: **kubemaster89296451**

    > [!TIP]
    > You can watch the deployment using the following command: `kubectl get pods -n kube-system -w`

> [!IMPORTANT]
> It can take around five to ten minutes for the OMS agents to begin submitting data to the Log Analytics workspace.
> You can view your Kubernetes cluster monitoring statistics via the [public Azure portal](https://aka.ms/azmon-containers).

#### Method Two

> [!IMPORTANT]
> This method **is used** by the [DeployAshKubernetes.ps1](linkhere) script.
> This method performs the [prerequisite step](#prerequisite-step) during the deployment.

This method utilises the [container-monitoring](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/containermonitoringaddon.md) add-on to deploy **Operations Management Suite (OMS) agent** containers to monitor the Kubernetes cluster on Azure Stack Hub.

To enable this functionality, provide the `-EnableMonitoring` switch when executing the [DeployAshKubernetes.ps1](linkhere) script.

> [!IMPORTANT]
> Once the cluster is deployed, it can take around five to ten minutes for the OMS agents to begin submitting data to the Log Analytics workspace.
> You can view your Kubernetes cluster monitoring statistics via the [public Azure portal](https://aka.ms/azmon-containers).

#### Kubernetes cluster dashboard

Provide the `-EnableDashboard` switch to the [DeployAshKubernetes.ps1](linkhere) script to be provided with access to the Kubernetes cluster dashboard.

## Deploying the AKS engine client VM and Kubernetes cluster

## [Linux](#tab/tabid-1)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$ArmEndpoint    | The Azure Resource Manager endpoint for Azure Stack Hub.                 | <form oninput="result.value=armendpoint.value" id="armendpoint" style="display: inline;"><input type="text" id="armendpoint" name="armendpoint" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$TenantId    | The tenant domain to login to.                                 | <form oninput="result.value=TenantId.value" id="TenantId" style="display: inline;"><input type="text" id="TenantId" name="TenantId" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl.value" id="tenantportalurl" style="display: inline;"><input type="text" id="tenantportalurl" name="tenantportalurl" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| \$AllowedIpAddresses   | A string array of IPs allowed to access the Kubernetes master node and client VM.          | <form oninput="result.value=AllowedIpAddresses.value" id="AllowedIpAddresses" style="display: inline;"><input type="text" id="AllowedIpAddresses" name="AllowedIpAddresses" style="display: inline;" placeholder='"127.0.0.1","192.168.0.1"'/></form> |
| \$ClusterResourceGroupName   | The resource group name to deploy the Kubernetes cluster into.         | <form oninput="result.value=clusterresourcegroupname.value" id="clusterresourcegroupname" style="display: inline;"><input type="text" id="clusterresourcegroupname" name="clusterresourcegroupname" style="display: inline;" placeholder="kube-rg"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with.                                | <form oninput="result.value=masternodecount.value" id="masternodecount" style="display: inline;"><input type="text" id="masternodecount" name="masternodecount" style="display: inline;" placeholder="3"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount.value" id="workernodecount" style="display: inline;"><input type="text" id="workernodecount" name="workernodecount" style="display: inline;" placeholder="3"/></form> |
| \$KubernetesVersion       | The version of Kubernetes to deploy using AKS engine.                   | <form oninput="result.value=kubernetesversion.value" id="kubernetesversion" style="display: inline;"><input type="text" id="kubernetesversion" name="kubernetesversion" style="display: inline;" placeholder="1.16.9"/></form> |
| -EnableMonitoring (switch) | Enable Azure Monitor for containers and ContainerInsights for a Log Analytics Workspace.       | <form onchange="result.value=enablemonitoring.value" id="enablemonitoring" style="display: inline;"><select name="enablemonitoring" id="enablemonitoring" style="display: inline;"><option value=" -EnableMonitoring">Enable</option><option value="">Disable</option></select></form> |
| \$LogAnalyticsWorkspaceName   | The name of the Log Analytics Workspace.        | <form oninput="result.value=loganalyticsworkspacename1.value" id="loganalyticsworkspacename1" style="display: inline;"><input type="text" id="loganalyticsworkspacename1" name="loganalyticsworkspacename1" style="display: inline;" placeholder="kubernetes-cluster-workspace"/></form> |
| \$LogAnalyticsWorkspaceResourceGroupName   | The name of the resource group which the Log Analytics Workspace resides in.        | <form oninput="result.value=loganalyticsworkspaceresourcegroupname1.value" id="loganalyticsworkspaceresourcegroupname1" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupname1" name="loganalyticsworkspaceresourcegroupname1" style="display: inline;" placeholder="kubernetes-cluster-workspace-rg"/></form> |
| \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics Workspace.                 | <form oninput="result.value=loganalyticsworkspacelocation1.value" id="loganalyticsworkspacelocation1" style="display: inline;"><input type="text" id="loganalyticsworkspacelocation1" name="loganalyticsworkspacelocation1" style="display: inline;" placeholder="UK South"/></form> |
| -EnableDashboard (switch) | Enable the Kubernetes cluster dashboard.       | <form onchange="result.value=enabledashboard.value" id="enabledashboard" style="display: inline;"><select name="enabledashboard" id="enabledashboard" style="display: inline;"><option value=" -EnableDashboard">Enable</option><option value="">Disable</option></select></form> |

1. Clone the Github repo using the following command:

    ```bash
    git clone x; cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

    <pre><code class="language-PowerShell">
    .\DeployAshKubernetes.ps1 -TenantId "<output form="TenantId" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -ArmEndpoint "<output form="armendpoint" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>" -AllowedIpAddresses <output form="AllowedIpAddresses" name="result" style="display: inline;">"127.0.0.1","192.168.0.1"</output> -ClusterResourceGroupName "<output form="clusterresourcegroupname" name="result" style="display: inline;">kube-rg</output>" -MasterNodeCount <output form="masternodecount" name="result" style="display: inline;">3</output> -WorkerNodeCount <output form="workernodecount" name="result" style="display: inline;">3</output> -KubernetesVersion "<output form="kubernetesversion" name="result" style="display: inline;">1.16.9</output>"<output form="enabledashboard" name="result" style="display: inline;"> -EnableDashboard</output><output form="enablemonitoring" name="result" style="display: inline;"> -EnableMonitoring</output> -LogAnalyticsWorkspaceName "<output form="loganalyticsworkspacename1" name="result" style="display: inline;">kubernetes-cluster-workspace</output>" -LogAnalyticsWorkspaceResourceGroupName "<output form="loganalyticsworkspaceresourcegroupname1" name="result" style="display: inline;">kubernetes-cluster-workspace-rg</output>" -LogAnalyticsWorkspaceLocation "<output form="loganalyticsworkspacelocation1" name="result" style="display: inline;">UK South</output>" -Verbose
    </code></pre>

## [Windows](#tab/tabid-2)

### Declare variables

Enter details below to provide values for the variables in the following command in this article:

| Variable name   | Variable description                                               | Input            |
|-----------------|--------------------------------------------------------------------|------------------|
| \$ArmEndpoint    | The Azure Resource Manager endpoint for Azure Stack Hub.                 | <form oninput="result.value=armendpoint1.value" id="armendpoint1" style="display: inline;"><input type="text" id="armendpoint1" name="armendpoint1" style="display: inline;" placeholder="https://management.frn00006.azure.ukcloud.com"/></form> |
| \$TenantId    | The tenant domain to login to.                                 | <form oninput="result.value=TenantId1.value" id="TenantId1" style="display: inline;"><input type="text" id="TenantId1" name="TenantId1" style="display: inline;" placeholder="contoso.onmicrosoft.com"/></form> |
| \$TenantPortalUrl   | The URL of the Azure Stack Hub tenant portal.          | <form oninput="result.value=tenantportalurl1.value" id="tenantportalurl1" style="display: inline;"><input type="text" id="tenantportalurl1" name="tenantportalurl1" style="display: inline;" placeholder="https://portal.frn00006.azure.ukcloud.com"/></form> |
| \$WindowsVMPassword     | The password used when deploying the Windows client VM. | <form oninput="result.value=windowsvmpassword.value" id="windowsvmpassword" style="display: inline;"><input type="text" id="windowsvmpassword" name="windowsvmpassword" style="display: inline;" placeholder="Password123!"/></form> |
| \$AllowedIpAddresses   | A string array of IPs allowed to access the Kubernetes master node and client VM.          | <form oninput="result.value=AllowedIpAddresses1.value" id="AllowedIpAddresses1" style="display: inline;"><input type="text" id="AllowedIpAddresses1" name="AllowedIpAddresses1" style="display: inline;" placeholder='"127.0.0.1","192.168.0.1"'/></form> |
| \$ClusterResourceGroupName   | The resource group name to deploy the Kubernetes cluster into.         | <form oninput="result.value=clusterresourcegroupname1.value" id="clusterresourcegroupname1" style="display: inline;"><input type="text" id="clusterresourcegroupname1" name="clusterresourcegroupname1" style="display: inline;" placeholder="kube-rg"/></form> |
| \$MasterNodeCount  | The number of master nodes to deploy the Kubernetes cluster with.                                | <form oninput="result.value=masternodecount1.value" id="masternodecount1" style="display: inline;"><input type="text" id="masternodecount1" name="masternodecount1" style="display: inline;" placeholder="3"/></form> |
| \$WorkerNodeCount       | The number of worker nodes to deploy the Kubernetes cluster with.                   | <form oninput="result.value=workernodecount1.value" id="workernodecount1" style="display: inline;"><input type="text" id="workernodecount1" name="workernodecount1" style="display: inline;" placeholder="3"/></form> |
| \$KubernetesVersion       | The version of Kubernetes to deploy using AKS engine.                   | <form oninput="result.value=kubernetesversion1.value" id="kubernetesversion1" style="display: inline;"><input type="text" id="kubernetesversion1" name="kubernetesversion1" style="display: inline;" placeholder="1.16.9"/></form> |
| -EnableMonitoring (switch) | Enable Azure Monitor for containers and ContainerInsights for a Log Analytics Workspace.       | <form onchange="result.value=enablemonitoring1.value" id="enablemonitoring1" style="display: inline;"><select name="enablemonitoring1" id="enablemonitoring1" style="display: inline;"><option value=" -EnableMonitoring">Enable</option><option value="">Disable</option></select></form> |
| \$LogAnalyticsWorkspaceName   | The name of the Log Analytics Workspace.        | <form oninput="result.value=loganalyticsworkspacename2.value" id="loganalyticsworkspacename2" style="display: inline;"><input type="text" id="loganalyticsworkspacename2" name="loganalyticsworkspacename2" style="display: inline;" placeholder="kubernetes-cluster-workspace"/></form> |
| \$LogAnalyticsWorkspaceResourceGroupName   | The name of the resource group which the Log Analytics Workspace resides in.        | <form oninput="result.value=loganalyticsworkspaceresourcegroupname2.value" id="loganalyticsworkspaceresourcegroupname2" style="display: inline;"><input type="text" id="loganalyticsworkspaceresourcegroupname2" name="loganalyticsworkspaceresourcegroupname2" style="display: inline;" placeholder="kubernetes-cluster-workspace-rg"/></form> |
| \$LogAnalyticsWorkspaceLocation    | The location of the Log Analytics Workspace.                 | <form oninput="result.value=loganalyticsworkspacelocation2.value" id="loganalyticsworkspacelocation2" style="display: inline;"><input type="text" id="loganalyticsworkspacelocation2" name="loganalyticsworkspacelocation2" style="display: inline;" placeholder="UK South"/></form> |
| -EnableDashboard (switch) | Enable the Kubernetes cluster dashboard.       | <form onchange="result.value=enabledashboard1.value" id="enabledashboard1" style="display: inline;"><select name="enabledashboard1" id="enabledashboard1" style="display: inline;"><option value=" -EnableDashboard">Enable</option><option value="">Disable</option></select></form> |


1. Clone the Github repo using the following command:

    ```bash
    git clone x; cd /path/to/dir
    ```

2. Execute the deployment script using the following command:

    <pre><code class="language-PowerShell">
    .\DeployAshKubernetes.ps1 -TenantId "<output form="TenantId1" name="result" style="display: inline;">contoso.onmicrosoft.com</output>" -ArmEndpoint "<output form="armendpoint1" name="result" style="display: inline;">https://management.frn00006.azure.ukcloud.com</output>" -TenantPortalUrl "<output form="tenantportalurl1" name="result" style="display: inline;">https://portal.frn00006.azure.ukcloud.com</output>" -Windows -WindowsVMPassword "<output form="windowsvmpassword" name="result" style="display: inline;">Password123!</output>" -AllowedIpAddresses <output form="AllowedIpAddresses1" name="result" style="display: inline;">"127.0.0.1","192.168.0.1"</output> -ClusterResourceGroupName "<output form="clusterresourcegroupname1" name="result" style="display: inline;">kube-rg</output>" -MasterNodeCount <output form="masternodecount1" name="result" style="display: inline;">3</output> -WorkerNodeCount <output form="workernodecount1" name="result" style="display: inline;">3</output> -KubernetesVersion "<output form="kubernetesversion1" name="result" style="display: inline;">1.16.9</output>"<output form="enabledashboard1" name="result" style="display: inline;"> -EnableDashboard</output><output form="enablemonitoring1" name="result" style="display: inline;"> -EnableMonitoring</output> -LogAnalyticsWorkspaceName "<output form="loganalyticsworkspacename2" name="result" style="display: inline;">kubernetes-cluster-workspace</output>" -LogAnalyticsWorkspaceResourceGroupName "<output form="loganalyticsworkspaceresourcegroupname2" name="result" style="display: inline;">kubernetes-cluster-workspace-rg</output>" -LogAnalyticsWorkspaceLocation "<output form="loganalyticsworkspacelocation2" name="result" style="display: inline;">UK South</output>" -Verbose
    </code></pre>

***

## Post Deployment

### Using SSH agent forwarding

A useful feature provided by SSH agent is agent forwarding. This allows you to forward your local SSH keys along the connection. This is useful as it means you only have to store the SSH private key on the client VM. SSH agent forwarding is utilised by using the `-A` parameter.

1. SSH or RDP to the client VM.

2. Add the **linuxprofile_rsa** SSH private key to the SSH agent using the following command: `ssh-add ~/.ssh/linuxprofile_rsa`

3. SSH to the Kubernetes master node using the following command: `ssh -A azureuser@masternodeip`

4. SSH from the Kubernetes master node to a worker node using the following command: `ssh -A azureuser@workernodeip`
