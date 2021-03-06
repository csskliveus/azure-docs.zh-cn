---
title: 快速入门：适用于 .NET 的表单识别器客户端库
description: 使用适用于 .NET 的表单识别器客户端库创建一个表单处理应用，该应用从自定义文档中提取键值对和表数据。
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: 31e1a0d912c6623f57d4ea256968102604ce42ff
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98132260"
---
> [!NOTE]
> 本指南使用 cURL 执行 REST API 调用。 还有[GitHub 上的示例代码](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/FormRecognizer/rest)，它说明了如何使用 Python 调用 REST API。

## <a name="prerequisites"></a>先决条件

* 已安装 [cURL](https://curl.haxx.se/windows/)。
* Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/cognitive-services/)
* 包含一组训练数据的 Azure 存储 Blob。 有关整理训练数据集的提示和选项，请参阅[为自定义模型生成训练数据集](../../build-training-data-set.md)。 对于本快速入门，可以使用[示例数据集](https://go.microsoft.com/fwlink/?linkid=2090451)（下载并提取 sample_data.zip）的“训练”文件夹下的文件。
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="创建表单识别器资源"  target="_blank">创建表单识别器资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到表单识别器 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
* 回执图像的 URL。 在本快速入门中，可以使用[示例图像](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/contoso-allinone.jpg)。
* 名片图像的 URL。 在本快速入门中，可以使用[示例图像](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/business_cards/business-card-english.jpg)。
* 账单图像的 URL。 在本快速入门中，可使用[示例文档](https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/forms/Invoice_1.pdf)。


## <a name="analyze-layout"></a>分析布局

可使用表单识别器来识别并提取文档中的表格、线条和单词，无需训练模型。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。
1. 将请求正文中的 URL 替换为其中一个示例 URL。

# <a name="v20"></a>[v2.0](#tab/v2-0)    

```bash
curl -v -X POST "https://{Endpoint}/formrecognizer/v2.0/layout/analyze"
-H "Content-Type: application/json"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
--data-ascii "{\"source\": \"http://example.com/test.jpg\"}" 
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    

```bash
curl -v -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.2/layout/analyze"
-H "Content-Type: application/json"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
--data-ascii "{\"source\": \"http://example.com/test.jpg\"}" 
```
---

你将收到 `202 (Success)` 响应，其中包含“Operation-Location”标头。 此标头的值包含一个可用于查询异步操作状态和获取结果的操作 ID。 在以下示例中，`analyzeResults/` 后面的字符串就是操作 ID。

```console
https://cognitiveservice/formrecognizer/v2.1-preview.2/layout/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-layout-results"></a>获取布局结果

调用[分析布局](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeLayoutAsync) API 后，可调用[获取分析布局结果](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeLayoutResult) API，来获取操作状态和已提取的数据 。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。
1. 将 `{resultId}` 替换为上一步中的操作 ID。

# <a name="v20"></a>[v2.0](#tab/v2-0)    

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/layout/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)  
```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/layout/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```
---

### <a name="examine-the-results"></a>检查结果

你将收到包含 JSON 内容的 `200 (success)` 响应。

请查看以下发票图像及其相应的 JSON 输出。 为了简单起见，已缩短输出。 `"readResults"` 节点包含每一行文本，以及其各自在页面上的边界框位置。 `"selectionMarks"` 节点（在 v2.1 预览版中）显示每个选择标记（复选框、单选框），并显示其状态是“已选中”还是“未选中”。 `"pageResults"` 字段显示表中的每个文本段，均带有行-列坐标。

:::image type="content" source="../../media/contoso-invoice.png" alt-text="包含表的 Contoso 项目声明文档。":::

# <a name="v20"></a>[v2.0](#tab/v2-0)    
```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:36:52Z",
    "lastUpdatedDateTime": "2020-08-20T20:36:58Z",
    "analyzeResult": {
        "version": "2.0.0",
        "readResults": [
            {
                "page": 1,
                "language": "en",
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/14/words/0",
                                    "#/readResults/0/lines/14/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)   
```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T20:40:50Z",
    "lastUpdatedDateTime": "2020-08-20T20:40:55Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": 0,
                "width": 8.5,
                "height": 11,
                "unit": "inch",
                "lines": [
                    {
                        "boundingBox": [
                            0.5826,
                            0.4411,
                            2.3387,
                            0.4411,
                            2.3387,
                            0.7969,
                            0.5826,
                            0.7969
                        ],
                        "text": "Contoso, Ltd.",
                        "words": [
                            {
                                "boundingBox": [
                                    0.5826,
                                    0.4411,
                                    1.744,
                                    0.4411,
                                    1.744,
                                    0.7969,
                                    0.5826,
                                    0.7969
                                ],
                                "text": "Contoso,",
                                "confidence": 1
                            },
                            {
                                "boundingBox": [
                                    1.8448,
                                    0.4446,
                                    2.3387,
                                    0.4446,
                                    2.3387,
                                    0.7631,
                                    1.8448,
                                    0.7631
                                ],
                                "text": "Ltd.",
                                "confidence": 1
                            }
                        ]
                    },
                    ...
                        ]
                    }
                ],
                "selectionMarks": [
                    {
                        "boundingBox": [
                            3.9737,
                            3.7475,
                            4.1693,
                            3.7475,
                            4.1693,
                            3.9428,
                            3.9737,
                            3.9428
                        ],
                        "confidence": 0.989,
                        "state": "selected"
                    },
                    ...
                ]
            }
        ],
        "pageResults": [
            {
                "page": 1,
                "tables": [
                    {
                        "rows": 5,
                        "columns": 5,
                        "cells": [
                            {
                                "rowIndex": 0,
                                "columnIndex": 0,
                                "text": "Training Date",
                                "boundingBox": [
                                    0.5133,
                                    4.2167,
                                    1.7567,
                                    4.2167,
                                    1.7567,
                                    4.4492,
                                    0.5133,
                                    4.4492
                                ],
                                "elements": [
                                    "#/readResults/0/lines/12/words/0",
                                    "#/readResults/0/lines/12/words/1"
                                ]
                            },
                            ...
                        ]
                    },
                    ...
                ]
            }
        ]
    }
}
``` 

---

## <a name="analyze-receipts"></a>分析回执

若要开始分析回执，可使用以下 cURL 命令调用[分析收据](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeReceiptAsync) API。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{your receipt URL}` 替换为回执图像的 URL 地址。
1. 将 `{subscription key>` 替换为从上一步复制的订阅密钥。

# <a name="v20"></a>[v2.0](#tab/v2-0)

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ \"source\": \"{your receipt URL}\"}"
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.2/prebuilt/receipt/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ \"source\": \"{your receipt URL}\"}"
```
---

你将收到 `202 (Success)` 响应，其中包含“Operation-Location”标头。 此标头的值包含一个可用于查询异步操作状态和获取结果的操作 ID。 在以下示例中，`operations/` 后面的字符串就是操作 ID。

```console
https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/receipt/operations/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-the-receipt-results"></a>获取回执结果

调用了“分析回执”API 后，可以调用 **[获取分析收据结果](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeReceiptResult)** API，以获取操作状态和已提取的数据。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获得的终结点。 可以在表单识别器资源的“概览”选项卡中找到该终结点。
1. 将 `{operationId}` 替换为上一步中的操作 ID。
1. 将 `{subscription key}` 替换为订阅密钥。

# <a name="v20"></a>[v2.0](#tab/v2-0)  
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)  
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/prebuilt/receipt/analyzeResults/{operationId}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
---

### <a name="examine-the-response"></a>检查响应

你将收到包含 JSON 输出的 `200 (Success)` 响应。 第一个字段 `"status"` 指示操作的状态。 如果操作已完成，`"readResults"` 字段包含已从回执中提取的每行文本，而 `"documentResults"` 字段包含该回执中最相关部分的键/值信息。 如果操作未完成，`"status"` 的值将为 `"running"` 或 `"notStarted"`，你应当采用手动方式或通过脚本再次调用该 API。 我们建议两次调用间隔一秒或更长时间。

请查看以下回执图像及其相应的 JSON 输出。 为可读起见，已将该输出缩短。

![Contoso 商店提供的回执](../../media/contoso-allinone.jpg)

`"readResults"` 节点包含所有已识别的文本（如果将可选 includeTextDetails 参数设置为 `true`）。 文本按页，然后按行，然后按单个单词进行组织。 `"documentResults"` 节点包含模型发现的特定于回执的值。 你将在这里找到有用的键/值对，如税款、总计、商家地址等。

```json
{
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{
    "version":"2.1.0",
    "readResults":[
      {
        "page":1,
        "angle":0.6893,
        "width":1688,
        "height":3000,
        "unit":"pixel",
        "language":"en",
        "lines":[
          {
            "text":"Contoso",
            "boundingBox":[
              635,
              510,
              1086,
              461,
              1098,
              558,
              643,
              604
            ],
            "words":[
              {
                "text":"Contoso",
                "boundingBox":[
                  639,
                  510,
                  1087,
                  461,
                  1098,
                  551,
                  646,
                  604
                ],
                "confidence":0.955
              }
            ]
          },
          ...
        ]
      }
    ],
    "documentResults":[
      {
        "docType":"prebuilt:receipt",
        "pageRange":[
          1,
          1
        ],
        "fields":{
          "ReceiptType":{
            "type":"string",
            "valueString":"Itemized",
            "confidence":0.692
          },
          "MerchantName":{
            "type":"string",
            "valueString":"Contoso Contoso",
            "text":"Contoso Contoso",
            "boundingBox":[
              378.2,
              292.4,
              1117.7,
              468.3,
              1035.7,
              812.7,
              296.3,
              636.8
            ],
            "page":1,
            "confidence":0.613,
            "elements":[
              "#/readResults/0/lines/0/words/0",
              "#/readResults/0/lines/1/words/0"
            ]
          },
          "MerchantAddress":{
            "type":"string",
            "valueString":"123 Main Street Redmond, WA 98052",
            "text":"123 Main Street Redmond, WA 98052",
            "boundingBox":[
              302,
              675.8,
              848.1,
              793.7,
              809.9,
              970.4,
              263.9,
              852.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/2/words/0",
              "#/readResults/0/lines/2/words/1",
              "#/readResults/0/lines/2/words/2",
              "#/readResults/0/lines/3/words/0",
              "#/readResults/0/lines/3/words/1",
              "#/readResults/0/lines/3/words/2"
            ]
          },
          "MerchantPhoneNumber":{
            "type":"phoneNumber",
            "valuePhoneNumber":"+19876543210",
            "text":"987-654-3210",
            "boundingBox":[
              278,
              1004,
              656.3,
              1054.7,
              646.8,
              1125.3,
              268.5,
              1074.7
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/4/words/0"
            ]
          },
          "TransactionDate":{
            "type":"date",
            "valueDate":"2019-06-10",
            "text":"6/10/2019",
            "boundingBox":[
              265.1,
              1228.4,
              525,
              1247,
              518.9,
              1332.1,
              259,
              1313.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[
              "#/readResults/0/lines/5/words/0"
            ]
          },
          "TransactionTime":{
            "type":"time",
            "valueTime":"13:59:00",
            "text":"13:59",
            "boundingBox":[
              541,
              1248,
              677.3,
              1261.5,
              668.9,
              1346.5,
              532.6,
              1333
            ],
            "page":1,
            "confidence":0.977,
            "elements":[
              "#/readResults/0/lines/5/words/1"
            ]
          },
          "Items":{
            "type":"array",
            "valueArray":[
              {
                "type":"object",
                "valueObject":{
                  "Quantity":{
                    "type":"number",
                    "text":"1",
                    "boundingBox":[
                      245.1,
                      1581.5,
                      300.9,
                      1585.1,
                      295,
                      1676,
                      239.2,
                      1672.4
                    ],
                    "page":1,
                    "confidence":0.92,
                    "elements":[
                      "#/readResults/0/lines/7/words/0"
                    ]
                  },
                  "Name":{
                    "type":"string",
                    "valueString":"Cappuccino",
                    "text":"Cappuccino",
                    "boundingBox":[
                      322,
                      1586,
                      654.2,
                      1601.1,
                      650,
                      1693,
                      317.8,
                      1678
                    ],
                    "page":1,
                    "confidence":0.923,
                    "elements":[
                      "#/readResults/0/lines/7/words/1"
                    ]
                  },
                  "TotalPrice":{
                    "type":"number",
                    "valueNumber":2.2,
                    "text":"$2.20",
                    "boundingBox":[
                      1107.7,
                      1584,
                      1263,
                      1574,
                      1268.3,
                      1656,
                      1113,
                      1666
                    ],
                    "page":1,
                    "confidence":0.918,
                    "elements":[
                      "#/readResults/0/lines/8/words/0"
                    ]
                  }
                }
              },
              ...
            ]
          },
          "Subtotal":{
            "type":"number",
            "valueNumber":11.7,
            "text":"11.70",
            "boundingBox":[
              1146,
              2221,
              1297.3,
              2223,
              1296,
              2319,
              1144.7,
              2317
            ],
            "page":1,
            "confidence":0.955,
            "elements":[
              "#/readResults/0/lines/13/words/1"
            ]
          },
          "Tax":{
            "type":"number",
            "valueNumber":1.17,
            "text":"1.17",
            "boundingBox":[
              1190,
              2359,
              1304,
              2359,
              1304,
              2456,
              1190,
              2456
            ],
            "page":1,
            "confidence":0.979,
            "elements":[
              "#/readResults/0/lines/15/words/1"
            ]
          },
          "Tip":{
            "type":"number",
            "valueNumber":1.63,
            "text":"1.63",
            "boundingBox":[
              1094,
              2479,
              1267.7,
              2485,
              1264,
              2591,
              1090.3,
              2585
            ],
            "page":1,
            "confidence":0.941,
            "elements":[
              "#/readResults/0/lines/17/words/1"
            ]
          },
          "Total":{
            "type":"number",
            "valueNumber":14.5,
            "text":"$14.50",
            "boundingBox":[
              1034.2,
              2617,
              1387.5,
              2638.2,
              1380,
              2763,
              1026.7,
              2741.8
            ],
            "page":1,
            "confidence":0.985,
            "elements":[
              "#/readResults/0/lines/19/words/0"
            ]
          }
        }
      }
    ]
  }
}
```

## <a name="analyze-business-cards"></a>分析名片

# <a name="v20"></a>[v2.0](#tab/v2-0)  

> [!IMPORTANT]
> 此功能在所选的 API 版本中不可用。

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)  

若要开始分析名片，请使用以下 cURL 命令调用[分析名片](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeBusinessCardAsync) API。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{your receipt URL}` 替换为回执图像的 URL 地址。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。

```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.2/prebuilt/businessCard/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ \"source\": \"{your receipt URL}\"}"
```

你将收到 `202 (Success)` 响应，其中包含“Operation-Location”标头。 此标头的值包含一个可用于查询异步操作状态和获取结果的操作 ID。

```console
https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/businessCard/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-business-card-results"></a>获取名片结果

调用分析名片 API 后，请调用[获取分析名片结果](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeBusinessCardResult) API，以获取操作状态和已提取的数据 。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获得的终结点。 可以在表单识别器资源的“概览”选项卡中找到该终结点。
1. 将 `{resultId}` 替换为上一步中的操作 ID。
1. 将 `{subscription key}` 替换为订阅密钥。

```bash
curl -v -X GET "https://westcentralus.api.cognitive.microsoft.com/formrecognizer/v2.1-preview.2/prebuilt/businessCard/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>检查响应

你将收到包含 JSON 输出的 `200 (Success)` 响应。 `"readResults"` 节点包含所有已识别的文本。 文本按页，然后按行，然后按单个单词进行组织。 `"documentResults"` 节点包含模型发现的特定于名片的值。 在此，你可以找到有用的联系人信息，如公司名、名字、姓氏、电话号码等。

![Contoso 公司的名片](../../media/business-card-english.jpg)

此示例说明表单识别器返回的 JSON 输出。 为了提高可读性，此响应已被截断。

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-06-04T08:19:29Z",
    "lastUpdatedDateTime": "2020-06-04T08:19:35Z",
    "analyzeResult": {
        "version": "2.1.1",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel"
            }
        ],
        "documentResults": [
            {
                "docType": "prebuilt:businesscard",
                "pageRange": [
                    1,
                    1
                ],
                "fields": {
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Avery",
                                        "text": "Avery",
                                        "boundingBox": [
                                            703,
                                            1096,
                                            1134,
                                            989,
                                            1165,
                                            1109,
                                            733,
                                            1206
                                        ],
                                        "page": 1
                                },
                                "text": "Dr. Avery Smith",
                                "boundingBox": [
                                    419.3,
                                    1154.6,
                                    1589.6,
                                    877.9,
                                    1618.9,
                                    1001.7,
                                    448.6,
                                    1278.4
                                ],
                                "confidence": 0.993
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "avery.smith@contoso.com",
                                "text": "avery.smith@contoso.com",
                                "boundingBox": [
                                    2107,
                                    934,
                                    2917,
                                    696,
                                    2935,
                                    764,
                                    2126,
                                    995
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Websites": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "https://www.contoso.com/",
                                "text": "https://www.contoso.com/",
                                "boundingBox": [
                                    2121,
                                    1002,
                                    2992,
                                    755,
                                    3014,
                                    826,
                                    2143,
                                    1077
                                ],
                                "page": 1,
                                "confidence": 0.995
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

脚本会向控制台输出响应，直到“分析名片”操作完成。

---

## <a name="analyze-invoices"></a>分析发票

# <a name="version-20"></a>[版本 2.0](#tab/v2-0)

> [!IMPORTANT]
> 此功能在所选的 API 版本中不可用。

# <a name="version-21-preview"></a>[版本 2.1 预览](#tab/v2-1)

若要开始分析账单，请使用以下 cURL 命令脚本调用[分析账单](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/5ed8c9843c2794cbb1a96291) API。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{your invoice URL}` 替换为账单文档的 URL 地址。
1. 将 `{subscription key}` 替换为订阅密钥。

```bash
curl -v -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.2/prebuilt/invoice/analyze"
-H "Content-Type: application/json"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
--data-ascii "{ \"source\": \"{your invoice URL}\"}"
```

你将收到 `202 (Success)` 响应，其中包含“Operation-Location”标头。 此标头的值包含一个可用于查询异步操作状态和获取结果的操作 ID。

```console
https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/invoice/analyzeResults/54f0b076-4e38-43e5-81bd-b85b8835fdfb
```

### <a name="get-invoice-results"></a>获取账单结果

调用分析账单 API 后，可调用[获取分析账单结果](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/5ed8c9acb78c40a2533aee83) API，来获取操作状态和已提取的数据 。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获得的终结点。 可以在表单识别器资源的“概览”选项卡中找到该终结点。
1. 将 `{resultId}` 替换为上一步中的操作 ID。
1. 将 `{subscription key}` 替换为订阅密钥。

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/prebuilt/invoice/analyzeResults/{resultId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

### <a name="examine-the-response"></a>检查响应

你将收到包含 JSON 输出的 `200 (Success)` 响应。 `"readResults"` 字段包含从账单提取的每行文本，`"pageResults"` 包含从账单提取的表格和选择标记，而 `"documentResults"` 字段包含账单中最相关部分的键/值信息。

请查看下列账单文档及其对应的 JSON 输出。 为便于阅读，已将该 JSON 内容缩短。

* [示例发票](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-invoice.pdf)

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-11-06T23:32:11Z",
    "lastUpdatedDateTime": "2020-11-06T23:32:20Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [{
            "page": 1,
            "angle": 0,
            "width": 8.5,
            "height": 11,
            "unit": "inch"
        }],
        "pageResults": [{
            "page": 1,
            "tables": [{
                "rows": 3,
                "columns": 4,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "QUANTITY",
                    "boundingBox": [0.4953,
                    5.7306,
                    1.8097,
                    5.7306,
                    1.7942,
                    6.0122,
                    0.4953,
                    6.0122]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "DESCRIPTION",
                    "boundingBox": [1.8097,
                    5.7306,
                    5.7529,
                    5.7306,
                    5.7452,
                    6.0122,
                    1.7942,
                    6.0122]
                },
                ...
                ],
                "boundingBox": [0.4794,
                5.7132,
                8.0158,
                5.714,
                8.0118,
                6.5627,
                0.4757,
                6.5619]
            },
            {
                "rows": 2,
                "columns": 6,
                "cells": [{
                    "rowIndex": 0,
                    "columnIndex": 0,
                    "text": "SALESPERSON",
                    "boundingBox": [0.4979,
                    4.963,
                    1.8051,
                    4.963,
                    1.7975,
                    5.2398,
                    0.5056,
                    5.2398]
                },
                {
                    "rowIndex": 0,
                    "columnIndex": 1,
                    "text": "P.O. NUMBER",
                    "boundingBox": [1.8051,
                    4.963,
                    3.3047,
                    4.963,
                    3.3124,
                    5.2398,
                    1.7975,
                    5.2398]
                },
                ...
                ],
                "boundingBox": [0.4976,
                4.961,
                7.9959,
                4.9606,
                7.9959,
                5.5204,
                0.4972,
                5.5209]
            }]
        }],
        "documentResults": [{
            "docType": "prebuilt:invoice",
            "pageRange": [1,
            1],
            "fields": {
                "AmountDue": {
                    "type": "number",
                    "valueNumber": 610,
                    "text": "$610.00",
                    "boundingBox": [7.3809,
                    7.8153,
                    7.9167,
                    7.8153,
                    7.9167,
                    7.9591,
                    7.3809,
                    7.9591],
                    "page": 1,
                    "confidence": 0.875
                },
                "BillingAddress": {
                    "type": "string",
                    "valueString": "123 Bill St, Redmond WA, 98052",
                    "text": "123 Bill St, Redmond WA, 98052",
                    "boundingBox": [0.594,
                    4.3724,
                    2.0125,
                    4.3724,
                    2.0125,
                    4.7125,
                    0.594,
                    4.7125],
                    "page": 1,
                    "confidence": 0.997
                },
                "BillingAddressRecipient": {
                    "type": "string",
                    "valueString": "Microsoft Finance",
                    "text": "Microsoft Finance",
                    "boundingBox": [0.594,
                    4.1684,
                    1.7907,
                    4.1684,
                    1.7907,
                    4.2837,
                    0.594,
                    4.2837],
                    "page": 1,
                    "confidence": 0.998
                },
                ...                
            }
        }]
    }
}
```

---

## <a name="train-a-custom-model"></a>训练自定义模型

若要训练自定义模型，需要 Azure 存储 Blob 中的一组训练数据。 应至少有五个类型/结构与主要输入数据相同的已填写表单（PDF 文档和/或图像）。 或者，可以使用一个空表单和两个已填充表单。 空表单的文件名需要包括单词“empty”。 有关整理训练数据的提示和选项，请参阅[为自定义模型生成训练数据集](../../build-training-data-set.md)。

> [!NOTE]
> 可以使用标记数据功能来手动预先标记部分或全部训练数据。 这是一个更为复杂的过程，但会生成更好的经过训练的模型。 有关此功能的详细信息，请参阅概述的[通过标签进行训练](../../overview.md#train-with-labels)部分。

若要使用 Azure Blob 容器中的文档训练表单识别器模型，请运行下面的 cURL 命令来调用[训练自定义模型](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/TrainCustomModelAsync) API  。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。
1. 将 `{SAS URL}` 替换为 Azure Blob 存储容器的共享访问签名 (SAS) URL。 [!INCLUDE [get SAS URL](../sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="SAS URL 检索":::

# <a name="v20"></a>[v2.0](#tab/v2-0)    
```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.0/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ \"source\": \""{SAS URL}"\"}"
```
# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    
```bash
curl -i -X POST "https://{Endpoint}/formrecognizer/v2.1-preview.2/custom/models" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" --data-ascii "{ \"source\": \""{SAS URL}"\"}"
```

---


你将收到包含“Location”标头的 `201 (Success)` 响应  。 此标头的值是要训练的新模型的 ID。

### <a name="get-training-results"></a>获取训练结果

开始训练操作后，可以使用新操作[获取自定义模型](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/GetCustomModel)检查训练状态  。 将模型 ID 传递到此 API 调用以检查训练状态：

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获得的终结点。
1. 将 `{subscription key}` 替换为订阅密钥
1. 将 `{model ID}` 替换为在上一步骤收到的模型 ID


# <a name="v20"></a>[v2.0](#tab/v2-0)    
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/custom/models/{model ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
    
---

你将收到以下格式的 JSON 正文的 `200 (Success)` 响应。 请注意 `"status"` 字段。 完成训练后，将出现值 `"ready"`。 如果模型没有完成训练，则需要重新运行该命令以再次查询该服务。 我们建议两次调用间隔一秒或更长时间。

`"modelId"` 字段包含要训练的模型的 ID。 下一步骤需要用到此字段。

```json
{
  "modelInfo":{
    "status":"ready",
    "createdDateTime":"2019-10-08T10:20:31.957784",
    "lastUpdatedDateTime":"2019-10-08T14:20:41+00:00",
    "modelId":"1cfb372bab404ba3aa59481ab2c63da5"
  },
  "trainResult":{
    "trainingDocuments":[
      {
        "documentName":"invoices\\Invoice_1.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_2.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_3.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_4.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      },
      {
        "documentName":"invoices\\Invoice_5.pdf",
        "pages":1,
        "errors":[

        ],
        "status":"succeeded"
      }
    ],
    "errors":[

    ]
  },
  "keys":{
    "0":[
      "Address:",
      "Invoice For:",
      "Microsoft",
      "Page"
    ]
  }
}
```

## <a name="analyze-forms-with-a-custom-model"></a>使用自定义模型分析表单

接下来，使用新的经过训练的模型分析文档并从中提取键值对和表。 运行以下 cURL 命令来调用[分析表单](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm) API  。 运行该命令之前，请进行以下更改：

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获取的终结点。 可以在表单识别器资源的“概览”选项卡中找到该终结点。 
1. 将 `{model ID}` 替换为在上一部分收到的模型 ID。
1. 将 `{SAS URL}` 替换为指向 Azure 存储中文件的 SAS URL。 按照“训练”部分中的步骤操作，但不是获取整个 blob 容器的 SAS URL，而是获取要分析的特定文件的 URL。
1. 将 `{subscription key}` 替换为订阅密钥。

# <a name="v20"></a>[v2.0](#tab/v2-0)    

```bash
curl -v "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ \"source\": \""{SAS URL}"\" } "
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    
```bash
curl -v "https://{Endpoint}/formrecognizer/v2.1-preview.2/custom/models/{model ID}/analyze" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {subscription key}" -d "{ \"source\": \""{SAS URL}"\" } "
```
    
---



你将收到 `202 (Success)` 响应，其中包含“Operation-Location”标头  。 此标头的值包括用于跟踪“分析”操作结果的结果 ID。 保存此结果 ID 以供下一步使用。

### <a name="get-the-analyze-results"></a>获取分析结果

使用以下 API 查询“分析”操作的结果。

1. 将 `{Endpoint}` 替换为从表单识别器订阅密钥中获取的终结点。 可以在表单识别器资源的“概览”选项卡中找到该终结点。 
1. 将 `{result ID}` 替换为上一部分中收到的 ID。
1. 将 `{subscription key}` 替换为订阅密钥。

# <a name="v20"></a>[v2.0](#tab/v2-0)    
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    
```bash
curl -X GET "https://{Endpoint}/formrecognizer/v2.1-preview/custom/models/{model ID}/analyzeResults/{result ID}" -H "Ocp-Apim-Subscription-Key: {subscription key}"
```
---

你将收到以下格式的 JSON 正文的 `200 (Success)` 响应。 为了简单起见，已缩短输出。 请注意底部附近的 `"status"` 字段。 完成“分析”操作时，会出现 `"succeeded"` 值。 如果“分析”操作尚未完成，则需要重新运行该命令以再次查询该服务。 我们建议两次调用间隔一秒或更长时间。

主键/值对关联和表位于 `"pageResults"` 节点中。 如果还通过 includeTextDetails URL 参数指定了纯文本提取，则 `"readResults"` 节点将显示文档中所有文本的内容和位置  。

为了简单起见，已缩短此示例 JSON 输出。

# <a name="v20"></a>[v2.0](#tab/v2-0)
```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T00:46:25Z",
  "lastUpdatedDateTime": "2020-08-21T00:46:32Z",
  "analyzeResult": {
    "version": "2.0.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0153,
              0.275,
              8.0944,
              0.275,
              8.0944,
              0.7125,
              5.0153,
              0.7125
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0153,
                  0.275,
                  6.2278,
                  0.275,
                  6.2278,
                  0.7125,
                  5.0153,
                  0.7125
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3292,
                  0.275,
                  8.0944,
                  0.275,
                  8.0944,
                  0.7125,
                  6.3292,
                  0.7125
                ]
              }
            ]
          }, 
        ...
        ]
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9722,
                1.0264,
                7.3417,
                1.0264,
                7.3417,
                1.1931,
                6.9722,
                1.1931
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "confidence": 1
          },
         ...
        ],
        "tables": [
          {
            "rows": 4,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6931,
                  4.2444,
                  1.5681,
                  4.2444,
                  1.5681,
                  4.4125,
                  0.6931,
                  4.4125
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ], 
        "clusterId": 0
      }
    ],
    "documentResults": [],
    "errors": []
  }
}
```    
# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    
```JSON
{
  "status": "succeeded",
  "createdDateTime": "2020-08-21T01:13:28Z",
  "lastUpdatedDateTime": "2020-08-21T01:13:42Z",
  "analyzeResult": {
    "version": "2.1.0",
    "readResults": [
      {
        "page": 1,
        "angle": 0,
        "width": 8.5,
        "height": 11,
        "unit": "inch",
        "lines": [
          {
            "text": "Project Statement",
            "boundingBox": [
              5.0444,
              0.3613,
              8.0917,
              0.3613,
              8.0917,
              0.6718,
              5.0444,
              0.6718
            ],
            "words": [
              {
                "text": "Project",
                "boundingBox": [
                  5.0444,
                  0.3587,
                  6.2264,
                  0.3587,
                  6.2264,
                  0.708,
                  5.0444,
                  0.708
                ]
              },
              {
                "text": "Statement",
                "boundingBox": [
                  6.3361,
                  0.3635,
                  8.0917,
                  0.3635,
                  8.0917,
                  0.6396,
                  6.3361,
                  0.6396
                ]
              }
            ]
          }, 
          ...
        ] 
      }
    ],
    "pageResults": [
      {
        "page": 1,
        "keyValuePairs": [
          {
            "key": {
              "text": "Date:",
              "boundingBox": [
                6.9833,
                1.0615,
                7.3333,
                1.0615,
                7.3333,
                1.1649,
                6.9833,
                1.1649
              ],
              "elements": [
                "#/readResults/0/lines/2/words/0"
              ]
            },
            "value": {
              "text": "9/10/2020",
              "boundingBox": [
                7.3833,
                1.0802,
                7.925,
                1.0802,
                7.925,
                1.174,
                7.3833,
                1.174
              ],
              "elements": [
                "#/readResults/0/lines/3/words/0"
              ]
            },
            "confidence": 1
          },
          ...
        ], 
        "tables": [
          {
            "rows": 5,
            "columns": 5,
            "cells": [
              {
                "text": "Training Date",
                "rowIndex": 0,
                "columnIndex": 0,
                "boundingBox": [
                  0.6944,
                  4.2779,
                  1.5625,
                  4.2779,
                  1.5625,
                  4.4005,
                  0.6944,
                  4.4005
                ],
                "confidence": 1,
                "rowSpan": 1,
                "columnSpan": 1,
                "elements": [
                  "#/readResults/0/lines/15/words/0",
                  "#/readResults/0/lines/15/words/1"
                ],
                "isHeader": true,
                "isFooter": false
              },
              ...
            ]
          }
        ], 
        "clusterId": 0
      }
    ], 
    "documentResults": [],
    "errors": []
  }
}
``` 
---

### <a name="improve-results"></a>改进结果

[!INCLUDE [improve results](../improve-results-unlabeled.md)]

## <a name="manage-custom-models"></a>管理自定义模型

### <a name="get-a-list-of-custom-models"></a>获取自定义模型列表

使用以下命令返回一个列表，其中列出属于你的订阅的所有自定义模型。

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。

# <a name="v20"></a>[v2.0](#tab/v2-0)    
```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/custom/models?op=full"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
```
---

你将收到一个 `200` 成功响应，其中包含如下所示的 JSON 数据。 `"modelList"` 元素包含所有已创建的模型及其信息。

```json
{
  "summary": {
    "count": 0,
    "limit": 0,
    "lastUpdatedDateTime": "string"
  },
  "modelList": [
    {
      "modelId": "string",
      "status": "creating",
      "createdDateTime": "string",
      "lastUpdatedDateTime": "string"
    }
  ],
  "nextLink": "string"
}
```

### <a name="get-a-specific-model"></a>获取特定模型

若要检索特定自定义模型的详细信息，请使用以下命令。

1. 将 `{Endpoint}` 替换为从表单识别器订阅中获取的终结点。
1. 将 `{subscription key}` 替换为从上一步复制的订阅密钥。
1. 将 `{modelId}` 替换为要查找的自定义模型的 ID。

# <a name="v20"></a>[v2.0](#tab/v2-0)    

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.0/custom/models/{modelId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
--data-ascii "{body}" 
```

# <a name="v21-preview"></a>[v2.1 预览版](#tab/v2-1)    

```bash
curl -v -X GET "https://{Endpoint}/formrecognizer/v2.1-preview.2/custom/models/{modelId}"
-H "Ocp-Apim-Subscription-Key: {subscription key}"
--data-ascii "{body}" 
```
---

你将收到一个 `200` 成功响应，其中包含如下所示的 JSON 数据。

```json
{
  "modelInfo": {
    "modelId": "string",
    "status": "creating",
    "createdDateTime": "string",
    "lastUpdatedDateTime": "string"
  },
  "keys": {
    "clusters": {}
  },
  "trainResult": {
    "trainingDocuments": [
      {
        "documentName": "string",
        "pages": 0,
        "errors": [
          "string"
        ],
        "status": "succeeded"
      }
    ],
    "fields": [
      {
        "fieldName": "string",
        "accuracy": 0.0
      }
    ],
    "averageModelAccuracy": 0.0,
    "errors": [
      {
        "message": "string"
      }
    ]
  }
}
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已使用表单识别器 REST API 以不同的方式训练模型和分析表单。 接下来，请参阅参考文档来深入了解表单识别器 API。

> [!div class="nextstepaction"]
> [REST API 参考文档](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)

* [什么是表单识别器？](../../overview.md)
