---
 title: include file
 description: include file
 services: virtual-network
 author: asudbring
 ms.service: virtual-network
 ms.topic: include
 ms.date: 06/06/2023
 ms.author: allensu
 ms.custom: include file
---

## Create a virtual network and an Azure Bastion host

The following procedure creates a virtual network with a resource subnet, an Azure Bastion subnet, and a Bastion host:

1. In the portal, search for and select **Virtual networks**.

1. On the **Virtual networks** page, select **+ Create**.

1. On the **Basics** tab of **Create virtual network**, enter or select the following information:

    | Setting | Value |
    |---|---|
    | **Project details** |  |
    | Subscription | Select your subscription. |
    | Resource group | Select **Create new**. </br> Enter **test-rg** for the name. </br> Select **OK**. |
    | **Instance details** |  |
    | Name | Enter **vnet-1**. |
    | Region | Select **East US 2**. |

    :::image type="content" source="./media/virtual-network-create-with-bastion/create-virtual-network-basics.png" alt-text="Screenshot of the Basics tab for creating a virtual network in the Azure portal.":::

1. Select **Next** to proceed to the **Security** tab.

1. In the **Azure Bastion** section, select **Enable Bastion**.

    Bastion uses your browser to connect to VMs in your virtual network over Secure Shell (SSH) or Remote Desktop Protocol (RDP) by using their private IP addresses. The VMs don't need public IP addresses, client software, or special configuration. For more information, see [What is Azure Bastion?](../articles/bastion/bastion-overview.md).

    > [!NOTE]
    > [!INCLUDE [Pricing](bastion-pricing.md)]

1. In **Azure Bastion**, enter or select the following information:

    | Setting | Value |
    |---|---|
    | Azure Bastion host name | Enter **bastion**. |
    | Azure Bastion public IP address | Select **Create a public IP address**. </br> Enter **public-ip-bastion** in Name. </br> Select **OK**. |

    :::image type="content" source="./media/virtual-network-create-with-bastion/enable-bastion.png" alt-text="Screenshot of options for enabling an Azure Bastion host as part of creating a virtual network in the Azure portal.":::

1. Select **Next** to proceed to the **IP Addresses** tab.

1. In the address space box in **Subnets**, select the **default** subnet.

1. In **Edit subnet**, enter or select the following information:

    | Setting | Value |
    |---|---|
    | **Subnet details** |  |
    | Subnet template | Leave the default of **Default**. |
    | Name | Enter **subnet-1**. |
    | Starting address | Leave the default of **10.0.0.0**. |
    | Subnet size | Leave the default of **/24 (256 addresses)**. |

    :::image type="content" source="./media/virtual-network-create-with-bastion/address-subnet-space.png" alt-text="Screenshot of configuration details for a subnet.":::

1. Select **Save**.

1. Select **Review + create** at the bottom of the window. When validation passes, select **Create**.
