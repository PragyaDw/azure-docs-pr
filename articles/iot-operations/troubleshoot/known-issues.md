---
title: "Known issues: Azure IoT Operations Preview"
description: A list of known issues for Azure IoT Operations.
author: dominicbetts
ms.author: dobett
ms.topic: troubleshooting-known-issue
ms.custom:
  - ignite-2023
ms.date: 05/03/2024
---

# Known issues: Azure IoT Operations Preview

[!INCLUDE [public-preview-note](../includes/public-preview-note.md)]

This article contains known issues for Azure IoT Operations Preview.

## Azure IoT Operations Preview

- You must use the Azure CLI interactive login `az login`. If you don't, you might see an error such as _ERROR: AADSTS530003: Your device is required to be managed to access this resource_.

- Uninstalling K3s: When you uninstall k3s on Ubuntu by using the `/usr/local/bin/k3s-uninstall.sh` script, you might encounter an issue where the script gets stuck on unmounting the NFS pod. A workaround for this issue is to run the following command before you run the uninstall script: `sudo systemctl stop k3s`.

## Azure IoT MQ Preview

- You can only access the default deployment by using the cluster IP, TLS, and a service account token. Clients outside the cluster need extra configuration before they can connect.

- You can't update the Broker custom resource after the initial deployment. You can't make configuration changes to cardinality, memory profile, or disk buffer.

- You can't configure the size of a disk-backed buffer unless your chosen storage class supports it.

- Even though IoT MQ's [diagnostic service](../monitor/howto-configure-diagnostics.md) produces telemetry on its own topic, you might still get messages from the self-test when you subscribe to `#` topic.

- Some clusters that have slow Kubernetes API calls may result in selftest ping failures: `Status {Failed}. Probe failed: Ping: 1/2` from running `az iot ops check` command.

- You might encounter an error in the KafkaConnector StatefulSet event logs such as `Invalid value: "mq-to-eventhub-connector-<token>--connectionstring": must be no more than 63 characters`. Ensure your KafkaConnector name is of maximum 5 characters.

- You may encounter timeout errors in the Kafka connector and Event Grid connector logs. Despite this, the connector will continue to function and forward messages.

## Azure IoT Layered Network Management Preview

- If the Layered Network Management service isn't getting an IP address while running K3S on Ubuntu host, reinstall K3S without _trafeik ingress controller_ by using the `--disable=traefik` option.

    ```bash
    curl -sfL https://get.k3s.io | sh -s - --disable=traefik --write-kubeconfig-mode 644
    ```

    For more information, see [Networking | K3s](https://docs.k3s.io/networking#traefik-ingress-controller).

- If DNS queries aren't getting resolved to expected IP address while using [CoreDNS](../manage-layered-network/howto-configure-layered-network.md#configure-coredns) service running on child network level, upgrade to Ubuntu 22.04 and reinstall K3S.

## Azure IoT OPC UA Broker Preview

- All AssetEndpointProfiles in the cluster have to be configured with the same transport authentication certificate, otherwise the OPC UA Broker might exhibit random behavior. To avoid this issue when using transport authentication, configure all asset endpoints with the same thumbprint for the transport authentication certificate in the Azure IoT Operations (preview) portal.

- If you deploy an AssetEndpointProfile into the cluster and the OPC UA Broker can't connect to the configured endpoint on the first attempt, then the OPC UA Broker never retries to connect.

    As a workaround, first fix the connection problem. Then either restart all the pods in the cluster with pod names that start with "aio-opc-opc.tcp", or delete the AssetEndpointProfile and deploy it again.

## OPC PLC simulator

If you create an asset endpoint for the OPC PLC simulator, but the OPC PLC simulator isn't sending data to the IoT MQ broker, run the following command to set `autoAcceptUntrustedServerCertificates=true` for the asset endpoint:

```bash
ENDPOINT_NAME=<name-of-you-endpoint-here>
kubectl patch AssetEndpointProfile $ENDPOINT_NAME \
-n azure-iot-operations \
--type=merge \
-p '{"spec":{"additionalConfiguration":"{\"applicationName\":\"'"$ENDPOINT_NAME"'\",\"security\":{\"autoAcceptUntrustedServerCertificates\":true}}"}}'
```

> [!CAUTION]
> Don't use this configuration in production or pre-production environments. Exposing your cluster to the internet without proper authentication might lead to unauthorized access and even DDOS attacks.

You can patch all your asset endpoints with the following command:

```bash
ENDPOINTS=$(kubectl get AssetEndpointProfile -n azure-iot-operations --no-headers -o custom-columns=":metadata.name")
for ENDPOINT_NAME in `echo "$ENDPOINTS"`; do \
kubectl patch AssetEndpointProfile $ENDPOINT_NAME \
   -n azure-iot-operations \
   --type=merge \
   -p '{"spec":{"additionalConfiguration":"{\"applicationName\":\"'"$ENDPOINT_NAME"'\",\"security\":{\"autoAcceptUntrustedServerCertificates\":true}}"}}'; \
done
```

Update the OPC UA Broker cluster extension to accept untrusted server certificates with the following command:

```azurecli
az k8s-extension update --version 0.3.0-preview --name opc-ua-broker --release-train preview --cluster-name <CLUSTER_NAME> --resource-group <RESOURCE_GROUP> --cluster-type connectedClusters --auto-upgrade-minor-version false --config opcPlcSimulation.deploy=true --config opcPlcSimulation.autoAcceptUntrustedCertificates=true
```

> [!CAUTION]
> Don't use this configuration in production or pre-production environments. The configuration lowers the security level for the OPC PLC so that it accepts connections from any client without an explicit peer certificate trust operation.

If the OPC PLC simulator isn't sending data to the IoT MQ broker after you create a new asset, restart the OPC PLC simulator pod. The pod name looks like `aio-opc-opc.tcp-1-f95d76c54-w9v9c`. To restart the pod, use the `k9s` tool to kill the pod, or run the following command:

```bash
kubectl delete pod aio-opc-opc.tcp-1-f95d76c54-w9v9c -n azure-iot-operations
```

## Azure IoT Akri Preview

A sporadic issue might cause the handler to restart with the following error in the logs: `opcua@311 exception="System.IO.IOException: Failed to bind to address http://unix:/var/lib/akri/opcua-asset.sock: address already in use.`.

To work around this issue, use the following steps to update the **DaemonSet** specification:

1. Locate the **Target** custom resource provided by **orchestration.iotoperations.azure.com** that contains the deployment specifications for **aio-opc-asset-discovery**.
1. In the **aio-opc-asset-discovery** component of the target file, find the `spect.components.aio-opc-asset-discovery.properties.resource.spec.template.spec.containers.env` parameter.
1. Add the following environment variables:

```yml
- name: ASPNETCORE_URLS 
  value: http://+8443 
- name: POD_IP 
  valueFrom: 
    fieldRef: 
      fieldPath: "status.podIP" 
```

The final specification should look like the following example:

```yml
apiVersion: orchestrator.iotoperations.azure.com/v1 
kind: Target 
metadata: 
  name: <cluster-name>-target 
  namespace: azure-iot-operations 
spec: 
  displayName: <cluster-name>-target 
  scope: azure-iot-operations 
  topologies: 
  ...
  version: 1.0.0.0 
  components: 
    ... 
    - name: aio-opc-asset-discovery 
      type: yaml.k8s 
      properties: 
        resource: 
          apiVersion: apps/v1 
          kind: DaemonSet 
          metadata: 
            labels: 
              app.kubernetes.io/part-of: aio 
            name: aio-opc-asset-discovery 
          spec: 
            selector: 
              matchLabels: 
                name: aio-opc-asset-discovery 
            template: 
              metadata: 
                labels: 
                  app.kubernetes.io/part-of: aio 
                  name: aio-opc-asset-discovery 
              spec: 
                containers: 
                  - env: 
                      - name: ASPNETCORE_URLS 
                        value: http://+8443 
                      - name: POD_IP 
                        valueFrom: 
                          fieldRef: 
                            fieldPath: status.podIP 
                      - name: DISCOVERY_HANDLERS_DIRECTORY 
                        value: /var/lib/akri 
                      - name: AKRI_AGENT_REGISTRATION 
                        value: 'true' 
                    image: >- 
                      edgeappmodel.azurecr.io/opcuabroker/discovery-handler:0.4.0-preview.3 
                    imagePullPolicy: Always 
                    name: aio-opc-asset-discovery 
                    ports: ... 
                    resources: ...
                    volumeMounts: ...
                volumes: ...
```

## Azure IoT Operations Preview portal

To sign in to the Azure IoT Operations portal, you need a Microsoft Entra ID account with at least contributor permissions for the resource group that contains your **Kubernetes - Azure Arc** instance. You can't sign in with a Microsoft account (MSA). To create an account in your Azure tenant:

1. Sign in to the [Azure portal](https://portal.azure.com/) with the same tenant and user name that you used to deploy Azure IoT Operations.
1. In the Azure portal, navigate to the **Microsoft Entra ID** section, select **Users > +New user > Create new user**. Create a new user and make a note of the password, you need it to sign in later.
1. In the Azure portal, navigate to the resource group that contains your **Kubernetes - Azure Arc** instance. On the **Access control (IAM)** page, select **+Add > Add role assignment**.
1. On the **Add role assignment page**, select **Privileged administrator roles**. Then select **Contributor** and then select **Next**.
1. On the **Members** page, add your new user to the role.
1. Select **Review and assign** to complete setting up the new user.

You can now use the new user account to sign in to the [Azure IoT Operations](https://iotoperations.azure.com) portal.
