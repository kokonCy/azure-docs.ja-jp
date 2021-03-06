---
title: "Azure CLI スクリプト - Azure Database for PostgreSQL の作成 | Microsoft Docs"
description: "Azure CLI スクリプト サンプル - Azure Database for PostgreSQL サーバーを作成して、サーバー レベルのファイアウォール規則を構成します。"
services: postgresql
author: salonisonpal
ms.author: salonis
manager: jhubbard
editor: jasonwhowell
ms.service: postgresql-database
ms.custom: mvc
ms.devlang: azure-cli
ms.topic: sample
ms.date: 05/31/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 4f68f90c3aea337d7b61b43e637bcfda3c98f3ea
ms.openlocfilehash: 18700246b496d037464d86945b24ac59c8d88756
ms.contentlocale: ja-jp
ms.lasthandoff: 06/20/2017

---

# <a name="create-an-azure-database-for-postgresql-server-and-configure-a-firewall-rule-using-the-azure-cli"></a>Azure CLI を使った Azure Database for PostgreSQL サーバーの作成とファイアウォール規則の構成
このサンプル CLI スクリプトは、Azure Database for PostgreSQL サーバーを作成して、サーバー レベルのファイアウォール規則を構成するものです。 スクリプトが正常に実行されると、すべての Azure サービスと構成済み IP アドレスから PostgreSQL サーバーにアクセスできるようになります。

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

CLI をローカルにインストールして使用する場合、このトピックでは、Azure CLI バージョン 2.0 以降を実行していることが要件です。 バージョンを確認するには、`az --version` を実行します。 インストールまたはアップグレードする必要がある場合は、「[Azure CLI 2.0 のインストール]( /cli/azure/install-azure-cli)」を参照してください。 

## <a name="sample-script"></a>サンプル スクリプト
このサンプル スクリプトでは、強調表示された行を編集して、管理者のユーザー名とパスワードをカスタマイズします。
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/create-postgresql-server-and-firewall-rule/create-postgresql-server-and-firewall-rule.sh?highlight=15-16 "Azure Database for PostgreSQL、およびサーバーレベルのファイアウォール規則を作成します。")]

## <a name="clean-up-deployment"></a>デプロイのクリーンアップ
スクリプト サンプルの実行後は、次のコマンドを使用してリソース グループとすべての関連リソースを削除することができます。
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/create-postgresql-server-and-firewall-rule/delete-postgresql.sh "リソース グループを削除します。")]

## <a name="script-explanation"></a>スクリプトの説明
このスクリプトでは、次のコマンドを使用します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| **コマンド** | **メモ** |
|---|---|
| [az group create](/cli/azure/group#create) | すべてのリソースを格納するリソース グループを作成します。 |
| [az postgres server create](/cli/azure/postgres/server#create) | データベースをホストする PostgreSQL サーバーを作成します。 |
| [az postgres server firewall create](/cli/azure/postgres/server/firewall-rule#create) | 対象のサーバーとそこにあるデータベースに対し、入力した IP アドレス範囲からアクセスできるようにするファイアウォール規則を作成します。 |
| [az group delete](/cli/azure/group#delete) | 入れ子になったリソースすべてを含むリソース グループを削除します。 |

## <a name="next-steps"></a>次のステップ
- Azure CLI に関する詳細を読む: [Azure CLI ドキュメント](/cli/azure/overview)
- 他のスクリプトを試す: [Azure Database for PostgreSQL の Azure CLI サンプル](../sample-scripts-azure-cli.md)

