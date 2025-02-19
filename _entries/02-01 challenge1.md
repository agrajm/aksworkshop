---
sectionid: deploy
sectionclass: h2
title: Deploy Kubernetes with Azure Kubernetes Service (AKS)
parent-id: upandrunning
---

Azure has a managed Kubernetes service, AKS (Azure Kubernetes Service), we'll use this to easily deploy and standup a Kubernetes cluster.

### Tasks

#### Get the latest Kubernetes version available in AKS

{% collapsible %}

Get the latest available Kubernetes version in your preferred region into a bash variable. Replace `<region>` with the region of your choosing, for example `eastus`.

```sh
version=$(az aks get-versions -l <region> --query 'orchestrators[-1].orchestratorVersion' -o tsv)
```

{% endcollapsible %}

#### Create a Resource Group

> **Note** You don't need to create a resource group if you're using the lab environment. You can use the resource group created for you as part of the lab. To retrieve the resource group name in the managed lab environment, run `az group list`.

{% collapsible %}

```sh
az group create --name <resource-group> --location <region>
```

{% endcollapsible %}

#### Create the AKS cluster

**Task Hints**
* It's recommended to use the Azure CLI and the `az aks create` command to deploy your cluster. Refer to the docs linked in the Resources section, or run `az aks create -h` for details
* The size and number of nodes in your cluster is not critical but two or more nodes of `DS2_v2` or larger is recommended

> **Note** You can create AKS clusters that support the [cluster autoscaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler#about-the-cluster-autoscaler). However, please note that the AKS cluster autoscaler is a preview feature, and enabling it is a more involved process. AKS preview features are self-service and opt-in. Previews are provided to gather feedback and bugs from our community. However, they are not supported by Azure technical support. If you create a cluster, or add these features to existing clusters, that cluster is unsupported until the feature is no longer in preview and graduates to general availability (GA).

##### **Option 1:** Create an AKS cluster without the cluster autoscaler (recommended)

> **Note** If you're using the provided lab environment, you'll not be able to create the Log Analytics workspace required to enable monitoring while creating the cluster from the Azure Portal unless you manually create the workspace in your assigned resource group. Additionally, if you're running this on an Azure Pass, please add `--load-balancer-sku basic` to the flags, as the Azure Pass only supports the basic Azure Load Balancer.

  {% collapsible %}

  Create AKS using the latest version

  ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys
  ```

  {% endcollapsible %}

##### **Option 2 (*Preview*):** Create an AKS cluster with the cluster autoscaler

> **Note** This will not work in the lab environment. You can only do this on a subscription where you have access to enable preview features. Additionally, if you're running this on an Azure Pass, please add `--load-balancer-sku basic` to the flags, as the Azure Pass only supports the basic Azure Load Balancer.

  {% collapsible %}
 
  AKS clusters that support the cluster autoscaler must use virtual machine scale sets and run Kubernetes version *1.12.4* or later. Cluster autoscaler support is in preview. To opt in and create clusters that use this feature, first install the *aks-preview* Azure CLI extension using the `az extension add` command, as shown in the following example:

  ```sh
  az extension add --name aks-preview
  ```

  Use the `az aks create` command specifying the `--enable-cluster-autoscaler` parameter, and a node `--min-count` and `--max-count`.

  > **Note** During preview, you can't set a higher minimum node count than is currently set for the cluster. For example, if you currently have min count set to *1*, you can't update the min count to *3*.

   ```sh
  az aks create --resource-group <resource-group> \
    --name <unique-aks-cluster-name> \
    --location <region> \
    --kubernetes-version $version \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3
  ```

  {% endcollapsible %}

#### Ensure you can connect to the cluster using `kubectl`

**Task Hints**
* `kubectl` is the main command line tool you will using for working with Kubernetes and AKS. It is already installed in the Azure Cloud Shell
* Refer to the AKS docs which includes [a guide for connecting kubectl to your cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster) (Note. using the cloud shell you can skip the `install-cli` step).
* A good sanity check is listing all the nodes in your cluster `kubectl get nodes`.
* [This is a good cheat sheet](https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf) for kubectl.

{% collapsible %}

> **Note** `kubectl`, the Kubernetes CLI, is already installed on the Azure Cloud Shell.

Authenticate

```sh
az aks get-credentials --resource-group <resource-group> --name <unique-aks-cluster-name>
```

List the available nodes

```sh
kubectl get nodes
```

{% endcollapsible %}

> **Resources**
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough>
> * <https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create>
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal>
> * <https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster>
> * <https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf>
