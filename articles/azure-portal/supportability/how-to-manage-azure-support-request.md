---
title: 管理 Azure 支持请求
description: 介绍如何查看支持请求，发送消息，更改请求严重性级别，与 Azure 支持共享诊断信息，重新打开已关闭的支持请求，并上传文件。
tags: billing
ms.assetid: 86697fdf-3499-4cab-ab3f-10d40d3c1f70
ms.topic: how-to
ms.date: 12/14/2020
ms.openlocfilehash: 8110f87401da1352309fb55615093d49981c754d
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504808"
---
# <a name="manage-an-azure-support-request"></a>管理 Azure 支持请求

[创建 Azure 支持请求](how-to-create-azure-support-request.md)后，可以在[Azure 门户](https://portal.azure.com)中对其进行管理，本文将对此进行介绍。 还可使用 [Azure 支持票证 REST API](/rest/api/support) 以编程方式创建和管理请求。

## <a name="view-support-requests"></a>查看支持请求

转到 "**帮助 + 支持**  >   **所有支持请求**"，查看支持请求的详细信息和状态。

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="所有支持请求":::

在此页上，你可以搜索、筛选和排序支持请求。 选择支持请求以查看详细信息，包括其严重性以及与请求关联的任何消息。

## <a name="send-a-message"></a>发送消息

1. 在 " **所有支持请求** " 页上，选择支持请求。

1. 在 " **支持请求** " 页上，选择 " **新建消息**"。

1. 输入消息，然后选择 " **提交**"。

## <a name="change-the-severity-level"></a>更改严重性级别

> [!NOTE]
> 最大严重性级别取决于你的 [支持计划](https://azure.microsoft.com/support/plans)。
>

1. 在 " **所有支持请求** " 页上，选择支持请求。

1. 在 " **支持请求** " 页上，选择 " **更改**"。

    :::image type="content" source="media/how-to-manage-azure-support-request/change-severity.png" alt-text="更改支持请求严重性":::

1. Azure 门户显示了两个屏幕中的一个，具体取决于你的请求是否已分配给支持工程师：

    - 如果未分配您的请求，您将看到类似于下面的屏幕。 选择新的严重性级别，然后选择 " **更改**"。

        :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="选择新的严重性级别":::

    - 如果你的请求已分配，你会看到如下所示的屏幕。 选择 **"确定"**，然后创建 [新消息](#send-a-message) 以请求严重性级别的更改。

        :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="无法选择新的严重性级别":::

## <a name="share-diagnostic-information-with-azure-support"></a>与 Azure 支持共享诊断信息

创建支持请求时，默认情况下会选择 " **共享诊断信息** " 选项。 这允许 Azure 支持人员收集 Azure 资源中的 [诊断信息](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) ：

* 创建请求后，不能清除此选项。

* 如果在创建请求时清除了该选项，则可以在创建请求后选择它。

    1. 在 " **所有支持请求** " 页上，选择支持请求。
    
    1. 在 " **支持请求** " 页上，选择 " **授予权限**"，然后选择 **"是** " 和 **"确定"**。
    
        :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="授予诊断信息的权限":::

## <a name="upload-files"></a>上传文件

您可以使用 "文件上传" 选项来上载诊断文件或您认为与支持请求相关的任何其他文件。

1. 在 " **所有支持请求** " 页上，选择支持请求。

1. 在 " **支持请求** " 页上，浏览查找文件，然后选择 " **上传**"。 如果有多个文件，请重复该过程。

    :::image type="content" source="media/how-to-manage-azure-support-request/file-upload.png" alt-text="上传文件":::

### <a name="file-upload-guidelines"></a>文件上传指南

使用 "文件上传" 选项时，请遵循以下准则：

* 为保护隐私，请不要在上传内容中含入任何个人信息。
* 文件名不得超过 110 个字符。
* 不能上载多个文件。
* 文件大小不能超过 4 MB。
* 所有文件必须具有文件扩展名，如 *.docx* 或 *.xlsx*。 下表显示了允许上载的文件扩展名。

| 0-9, A-C     | D-G   | H-M         | N-P   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7z         | .dat  | har        | .odx  | .rar     | .tdb       | .xlam   |
| .a          | .db   | .hwl        | .oft  | .rdl     | .tdf       | .xlr    |
| .abc        | .DMP  | .ics        | .old  | .rdlc    | .text      | .xls    |
| .adm        | .do_  | .ini        | .one  | .re_     | .thmx      | .xlsb   |
| .aspx       | .doc  | .java       | .osd  | .reg     | .tif       | .xlsm   |
| .ATF        | .docm | .jpg        | .OUT  | .remove  | .trc       | .xlsx   |
| .b          | .docx | .LDF        | P1   | .ren     | .TTD       | .xlt    |
| .ba_        | .dotm | .letterhead | .pcap | .rename  | .tx_       | .xltx   |
| .bak        | .dotx | .lnk        | .pdb  | .rft     | .txt       | .xml    |
| .bat        | .dtsx | .lo_        | .pdf  | .rpt     | .uccapilog | .xmla   |
| .blg        | .eds  | .log        | .piz  | .rte     | .uccplog   | .xps    |
| .CA_        | .emf  | .lpk        | .pmls | .rtf     | .udcx      | .xsd    |
| .CAB        | .eml  | .manifest   | .png  | .run     | .vb_       | .xsn    |
| .cap        | .emz  | .master     | .potx | .saz     | .vbs_      | .xxx    |
| .catx       | .err  | .mdmp       | .ppt  | .sql     | .vcf       | .z_     |
| .CFG        | .etl  | .mof        | .pptm | .sqlplan | .vsd       | .z01    |
| .compressed | .evt  | .mp3        | .pptx | .stp     | .wdb       | .z02    |
| .Config     | .evtx | .mpg        | .prn  | .svclog  | .wks       | .zi     |
| .cpk        | .EX   | .ms_        | .psf  | -        | .wma       | .zi_    |
| .cpp        | .ex_  | .msg        | .pst  | -        | .wmv       | .zip    |
| .cs         | .ex0  | .msi        | .pub  | -        | .wmz       | .zip_   |
| .CSV        | .FRD  | .mso        | -     | -        | .wps       | .zipp   |
| .cvr        | .gif  | .msu        | -     | -        | .wpt       | .zipped |
| -           | .guid | .nfo        | -     | -        | .wsdl      | .zipped  |
| -           | .gz   | -           | -     | -        | .wsp       | .zipx   |
| -           | -     | -           | -     | -        | .wtl       | .zit    |
| -           | -     | -           | -     | -        | -          | .zix    |
| -           | -     | -           | -     | -        | -          | .zzz    |

## <a name="close-a-support-request"></a>关闭支持请求

如果需要关闭支持请求，请 [发送一条消息](#send-a-message) ，询问是否关闭请求。

## <a name="reopen-a-closed-request"></a>重新打开已关闭的请求

如果需要重新打开已关闭的支持请求，请创建一 [条新消息](#send-a-message)，该消息将自动重新打开请求。

## <a name="cancel-a-support-plan"></a>取消支持计划

如果需要取消支持计划，请参阅 [取消支持计划](../../cost-management-billing/manage/cancel-azure-subscription.md#cancel-a-support-plan)。

## <a name="next-steps"></a>后续步骤

[如何创建 Azure 支持请求](how-to-create-azure-support-request.md)

[Azure 支持票证 REST API](/rest/api/support)
