---
title: "GCPのapp-script以下の見えないアクティブなプロジェクトを削除する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Google Workspace", "gcp", "Google App Script]
published: true
---

GCPの `app-script` フォルダが削除できなくて困った経験はありませんか？  
アクティブなプロジェクトが残っていると、Google Workspaceも削除できなくて困った方はいるのではないでしょうか（っていうか私がそうです）

![フォルダ内にアクティブなリソースが含まれているため、フォルダを削除できません。削除できるのは、空のフォルダのみです。](/images/2022-11-18-08-03-12.png)

`app-script` フォルダの下には、Google App Scriptに対応するプロジェクトが存在します。これらは誤って削除されることを防ぐため、デフォルトで表示されていません。  
2通りの方法で削除できます。

## 方法1: CLIで削除する

ターミナルから `gcloud` コマンドで削除します。

```shell
% gcloud projects list --filter='parent.id=395902557168'    # 395902557168 は私の app-script フォルダのIDです。
PROJECT_ID                      NAME             PROJECT_NUMBER
sys-04625505137771139140396653         slack     89322811207
sys-51362071043164527944101788  convert-json     471832586993
sys-60008403636720525138286053                   645991437074

% gcloud projects delete sys-04625505137771139140396653
Your project will be deleted.

Do you want to continue (Y/n)?  Y

Deleted [https://cloudresourcemanager.googleapis.com/v1/projects/sys-04625505137771139140396653].

You can undo this operation for a limited period by running the command below.
    $ gcloud projects undelete sys-04625505137771139140396653

See https://cloud.google.com/resource-manager/docs/creating-managing-projects for information on shutting down projects.

% gcloud projects delete sys-51362071043164527944101788
Your project will be deleted.

Do you want to continue (Y/n)?  Y

Deleted [https://cloudresourcemanager.googleapis.com/v1/projects/sys-51362071043164527944101788].

You can undo this operation for a limited period by running the command below.
    $ gcloud projects undelete sys-51362071043164527944101788

See https://cloud.google.com/resource-manager/docs/creating-managing-projects for information on shutting down projects.

% gcloud projects delete sys-60008403636720525138286053
Your project will be deleted.

Do you want to continue (Y/n)?  Y

Deleted [https://cloudresourcemanager.googleapis.com/v1/projects/sys-60008403636720525138286053].

You can undo this operation for a limited period by running the command below.
    $ gcloud projects undelete sys-60008403636720525138286053

See https://cloud.google.com/resource-manager/docs/creating-managing-projects for information on shutting down projects.
```

### 参考

[Apps Script Cloud プロジェクトの削除](https://developers.google.com/apps-script/guides/cloud-platform-projects#delete)

## 方法2: 

1. [script.google.com](https://script.google.com/) に移動します。
2. 削除するプロジェクトの右側にあるその他アイコン more_vert > [削除] > [削除] をクリックします。

ただし、このやり方はGoogle WorkspaceのSubscribeをストップした後だと使えません。

### 参考

[スタンドアロン プロジェクトを削除する](https://developers.google.com/apps-script/guides/projects#delete_a_standalone_project)

## 管理画面から表示するには

`app-script`のプロジェクトは、そのままではリソース管理の一覧には表示されません。  
フィルタで親IDとして `app-script` のIDを指定して、初めて表示することができます。

![](/images/2022-11-18-07-57-25.png)

### 参考

[デフォルトの Cloud プロジェクトを表示または編集する](https://developers.google.com/apps-script/guides/admin/view-cloud-projects#view_or_edit_default_cloud_projects)

## まとめ

GCPやGoogle Workspaceの削除に必要な、`app-script`の削除方法のガイドをまとめました。  
ドキュメントに載っていることばかりですが、私は簡単にたどり着けなかったので、ご参考になれば幸いです。
