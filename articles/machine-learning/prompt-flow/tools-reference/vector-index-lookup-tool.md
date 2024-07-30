---
title: Vector Index Lookup tool in Azure Machine Learning prompt flow
titleSuffix: Azure Machine Learning
description: Vector Index Lookup is a tool tailored for querying within an Azure Machine Learning vector index. It empowers users to extract contextually relevant information from a domain knowledge base.
services: machine-learning
ms.service: machine-learning
ms.subservice: prompt-flow
ms.custom:
  - ignite-2023
ms.topic: reference
author: likebupt
ms.author: keli19
ms.reviewer: lagayhar
ms.date: 03/21/2024
---

# Vector Index Lookup

> [!IMPORTANT]
> Vector, Vector DB and Faiss Index Lookup tools are deprecated and will be retired soon. [Migrated to the new Index Lookup tool (preview).](index-lookup-tool.md#how-to-migrate-from-legacy-tools-to-the-index-lookup-tool)

Vector Index Lookup is a tool tailored for querying within an Azure Machine Learning vector index. It empowers users to extract contextually relevant information from a domain knowledge base.

## Prerequisites

- Follow the instructions from sample flow `Bring your own Data QnA` to prepare a vector index as an input.
- Based on where you put your vector index, the identity used by the prompt flow runtime should be granted with certain roles. See the [steps to assign an Azure role](../../../role-based-access-control/role-assignments-steps.md).

    | Location | Role |
    | ---- | ---- |
    | Workspace datastores or workspace default blob | AzureML Data Scientist |
    | Other blobs | Storage Blob Data Reader |

> [!NOTE]
> When legacy tools switch to code-first mode, if you encounter the error `embeddingstore.tool.vector_index_lookup.search' is not found`, see the [troubleshooting guidance](./troubleshoot-guidance.md).

## Inputs

The tool accepts the following inputs:

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| path | string | blob/AML asset/datastore URL for the VectorIndex<br><br>blob URL format:<br>https://`<account_name>`.blob.core.windows.net/`<container_name>`/`<path_and_folder_name>`.<br><br>Azure Machine Learning asset URL format:<br>azureml://subscriptions/`<your_subscription>`/resourcegroups/`<your_resource_group>>`/workspaces/`<your_workspace>`/data/`<asset_name and optional version/label>`<br><br>Machine Learning datastore URL format:<br>azureml://subscriptions/`<your_subscription>`/resourcegroups/`<your_resource_group>`/workspaces/`<your_workspace>`/datastores/`<your_datastore>`/paths/`<data_path>` | Yes |
| query | string, list[float] | Text to be queried<br>or<br>Target vector to be queried, which the LLM tool can generate. | Yes |
| top_k | integer | Count of top-scored entities to return. Default value is 3. | No |

## Outputs

The following example is for a JSON format response returned by the tool, which includes the top-k scored entities. The entity follows a generic schema of vector search result provided by promptflow-vectordb SDK. For the Vector Index Search, the following fields are populated:

| Field Name | Type | Description |
| ---- | ---- | ----------- |
| text | string | Text of the entity. |
| score | float | Depends on index type defined in the vector index. If the index type is Faiss, the score is L2 distance. If the index type is Azure AI Search, the score is cosine similarity. |
| metadata | dict | Customized key-value pairs provided by the user when creating the index. |
| original_entity | dict | Depends on index type defined in the vector index. The original response JSON from the search REST API.|
  
```json
[
  {
    "text": "sample text #1",
    "vector": null,
    "score": 0.0,
    "original_entity": null,
    "metadata": {
      "link": "http://sample_link_1",
      "title": "title1"
    }
  },
  {
    "text": "sample text #2",
    "vector": null,
    "score": 0.07032840698957443,
    "original_entity": null,
    "metadata": {
      "link": "http://sample_link_2",
      "title": "title2"
    }
  },
  {
    "text": "sample text #0",
    "vector": null,
    "score": 0.08912381529808044,
    "original_entity": null,
    "metadata": {
      "link": "http://sample_link_0",
      "title": "title0"
    }
  }
]
```

## Deploying to an online endpoint

When you deploy a flow containing the vector index lookup tool to an online endpoint, there's an extra step to set up permissions. During deployment through the web pages, there's a choice between System-assigned and User-assigned Identity types. Either way, using the Azure portal (or a similar functionality), add the "AzureML Data Scientist" role of Azure Machine learning studio to the identity assign to the endpoint.