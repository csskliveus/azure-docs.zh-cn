---
title: 使用 AzCopy v10 从 Azure Blob 存储下载 blob |Microsoft Docs
description: 本文包含一系列 AzCopy 示例命令，可帮助你从 Azure Blob 存储下载 blob。
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 12/11/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: 382adb36712fbf4bee83044c8b2d096223eb6269
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630031"
---
# <a name="download-blobs-from-azure-blob-storage-by-using-azcopy-v10"></a>使用 AzCopy v10 从 Azure Blob 存储下载 blob

可以使用 AzCopy v10 命令行实用程序从 Blob 存储下载 blob 和目录。 

若要查看其他类型任务的示例，例如上传文件、与 Blob 存储同步或在帐户之间复制 blob，请参阅本文 [后续步骤](#next-steps) 部分中提供的链接。

## <a name="get-started"></a>入门

请参阅 [AzCopy 入门](storage-use-azcopy-v10.md)一文下载 AzCopy，并了解如何提供存储服务的授权凭据。

> [!NOTE] 
> 本文中的示例假定你已通过使用 Azure Active Directory (Azure AD) 提供了授权凭据。
>
> 如果你希望使用 SAS 令牌来授权访问 Blob 数据，可将该令牌追加到每个 AzCopy 命令中的资源 URL。 例如：`'https://<storage-account-name>.blob.core.windows.net/<container-name><SAS-token>'`。

## <a name="download-a-blob"></a>下载 blob

使用 [azcopy copy](storage-ref-azcopy-copy.md) 命令下载 blob。

> [!TIP]
> 此示例将路径参数括在单引号 ('') 内。 在除 Windows 命令 Shell (cmd.exe) 以外的所有命令 shell 中，都请使用单引号。 如果使用 Windows 命令 Shell (cmd.exe)，请用双引号 ("") 而不是单引号 ('') 括住路径参数。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> 如果 Blob 的 `Content-md5` 属性值包含哈希，AzCopy 将计算已下载的数据的 MD5 哈希，并验证存储在该 Blob 的 `Content-md5` 属性中的 MD5 哈希是否与计算出的哈希相匹配。 如果这些值不匹配，除非通过将 `--check-md5=NoCheck` 或 `--check-md5=LogOnly` 追加到 copy 命令来重写此行为，否则下载将会失败。

## <a name="download-a-directory"></a>下载目录

使用 [azcopy copy](storage-ref-azcopy-copy.md) 命令下载目录。

> [!TIP]
> 此示例将路径参数括在单引号 ('') 内。 在除 Windows 命令 Shell (cmd.exe) 以外的所有命令 shell 中，都请使用单引号。 如果使用 Windows 命令 Shell (cmd.exe)，请用双引号 ("") 而不是单引号 ('') 括住路径参数。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<directory-path>' '<local-directory-path>' --recursive` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myBlobDirectory' 'C:\myDirectory'  --recursive` |

此示例将生成一个名为 `C:\myDirectory\myBlobDirectory` 的目录，其中包含所有已下载的 blob。

## <a name="download-directory-contents"></a>下载目录内容

可以使用通配符 (*) 下载目录的内容，而无需复制包含的目录本身。

> [!TIP]
> 此示例将路径参数括在单引号 ('') 内。 在除 Windows 命令 Shell (cmd.exe) 以外的所有命令 shell 中，都请使用单引号。 如果使用 Windows 命令 Shell (cmd.exe)，请用双引号 ("") 而不是单引号 ('') 括住路径参数。

> [!NOTE]
> 目前，只有不使用分层命名空间的帐户才支持此方案。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.blob.core.windows.net/<container-name>/*' '<local-directory-path>/'` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory/*' 'C:\myDirectory'` |

追加 `--recursive` 标志以下载所有子目录中的文件。

## <a name="download-specific-blobs"></a>下载特定 blob

您可以使用完整的文件名、包含通配符字符的部分名称 ( * ) 或使用日期和时间来下载特定 blob。 

> [!TIP]
> 这些示例用单引号将路径参数 ( "" ) 。 在除 Windows 命令 Shell (cmd.exe) 以外的所有命令 shell 中，都请使用单引号。 如果使用 Windows 命令 Shell (cmd.exe)，请用双引号 ("") 而不是单引号 ('') 括住路径参数。

#### <a name="specify-multiple-complete-blob-names"></a>指定多个完整的 blob 名称

结合 `--include-path` 选项使用 [azcopy copy](storage-ref-azcopy-copy.md) 命令。 使用 semicolin () 分离单独的 blob 名称 `;` 。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>'  --include-path <semicolon-separated-file-list>` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt' --recursive` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-path 'photos;documents\myFile.txt'--recursive` |

在此示例中，AzCopy 将传输 `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` 目录和 `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/documents/myFile.txt` 文件。 包含 `--recursive` 用于传输目录中所有 blob 的选项 `https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/photos` 。

还可以通过使用选项排除 blob `--exclude-path` 。 有关详细信息，请参阅 [azcopy copy](storage-ref-azcopy-copy.md) 参考文档。

#### <a name="use-wildcard-characters"></a>使用通配符

结合 `--include-pattern` 选项使用 [azcopy copy](storage-ref-azcopy-copy.md) 命令。 指定包含通配符的部分名称。 使用分号 (`;`) 分隔名称。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>' '<local-directory-path>' --include-pattern <semicolon-separated-file-list-with-wildcard-characters>` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory' 'C:\myDirectory'  --include-pattern 'myFile*.txt;*.pdf*'` |

还可以通过使用选项排除 blob `--exclude-pattern` 。 有关详细信息，请参阅 [azcopy copy](storage-ref-azcopy-copy.md) 参考文档。

`--include-pattern`和 `--exclude-pattern` 选项仅适用于 blob 名称，不适用于路径。  如果要复制目录树中存在的 (blob) 的所有文本文件，请使用 `–recursive` 选项获取整个目录树，并使用 `–include-pattern` 和指定 `*.txt` 获取所有文本文件。

#### <a name="download-blobs-that-were-modified-before-or-after-a-date-and-time"></a>下载在日期和时间之前或之后修改的 blob 

将 [azcopy copy](storage-ref-azcopy-copy.md) 命令与 `--include-before` 或选项一起使用 `--include-after` 。 以 ISO-8601 格式指定日期和时间（例如 `2020-08-19T15:04:00Z`）。 

下面的示例下载在指定日期或指定日期之后修改的文件。

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-or-directory-name>/*' '<local-directory-path>' --include-after <Date-Time-in-ISO-8601-format>` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/FileDirectory/*' 'C:\myDirectory'  --include-after '2020-08-19T15:04:00Z'` |

如需详细的参考，请查看 [azcopy copy](storage-ref-azcopy-copy.md) 参考文档。

#### <a name="download-previous-versions-of-a-blob"></a>下载以前版本的 blob

如果已启用 [blob 版本控制](../blobs/versioning-enable.md)，则可下载一个或多个以前版本的 blob。 

首先，创建一个包含 [版本 id](../blobs/versioning-overview.md)列表的文本文件。 每个版本 ID 必须出现在单独的行中。 例如： 

```
2020-08-17T05:50:34.2199403Z
2020-08-17T05:50:34.5041365Z
2020-08-17T05:50:36.7607103Z
```

然后，将 [azcopy copy](storage-ref-azcopy-copy.md) 命令与选项一起使用 `--list-of-versions` 。 指定包含版本列表的文本文件的位置 (例如： `D:\\list-of-versions.txt`) 。  

#### <a name="download-a-blob-snapshot"></a>下载 Blob 快照

可以通过引用 blob 快照的 **DateTime** 值下载 [blob 快照](/azure/storage/blobs/snapshots-overview)。 

|    |     |
|--------|-----------|
| **语法** | `azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>?sharesnapshot=<DateTime-of-snapshot>' '<local-file-path>'` |
| **示例** | `azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |
| **示例**（分层命名空间） | `azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt?sharesnapshot=2020-09-23T08:21:07.0000000Z' 'C:\myDirectory\myTextFile.txt'` |

> [!NOTE]
> 如果你使用 SAS 令牌来授予对 blob 数据的访问权限，则在 SAS 令牌后面追加快照 **日期时间** 。 例如：`'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D&sharesnapshot=2020-09-23T08:21:07.0000000Z'`。

## <a name="download-with-optional-flags"></a>下载可选标志

可以通过使用可选标志来调整下载操作。 下面是几个示例。

|方案|标志|
|---|---|
|自动解压缩文件。|**--decompress**|
|指定你希望与复制相关的日志条目达到何种详细程度。|**--log-level**=\[WARNING\|ERROR\|INFO\|NONE\]|
|指定是否以及如何覆盖目标位置的冲突文件和 Blob。|**--overwrite**=\[true\|false\|ifSourceNewer\|prompt\]|

有关完整列表，请参阅[选项](storage-ref-azcopy-copy.md#options)。

## <a name="next-steps"></a>后续步骤

在以下文章中查找更多示例：

- [示例：上载](storage-use-azcopy-blobs-upload.md)
- [示例：帐户之间的复制](storage-use-azcopy-blobs-copy.md)
- [示例：同步](storage-use-azcopy-blobs-synchronize.md)
- [示例：Amazon S3 存储桶](storage-use-azcopy-s3.md)
- [示例： Azure 文件](storage-use-azcopy-files.md)
- [教程：使用 AzCopy 将本地数据迁移到云存储](storage-use-azcopy-migrate-on-premises-data.md)
- [对 AzCopy 进行配置、优化和故障排除](storage-use-azcopy-configure.md)