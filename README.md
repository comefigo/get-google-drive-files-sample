# 説明

## フォルダとファイルの一覧の取得

objParamではファイル一覧を取得する際の各種パラメータをオブジェクトで指定しています。  
指定できるプロパティの一覧は[こちら](https://developers.google.com/drive/v3/reference/files/list)になります。  
fieldsは戻り値で含めてほしい項目として指定できます。
```nexPageToken```はpageSizeで指定した数以上にファイルまたはフォルダがあった際に、次ページの一覧を取得する際のトークンになります。  
```files(id, name, kind...)```は戻り値のfilesプロパティの配下の項目（ファイルのメタ情報の項目）を指定できます。  
指定できるファイルのメタ情報は[こちら](https://developers.google.com/drive/v3/reference/files)を参照してください。

```
// リクエストパラメータ
var objParam = {
    pageSize: 10,
    orderBy: 'folder,modifiedTime',
    q: 'trashed=false',
    fields: 'files(id, name, kind, size, mimeType, lastModifyingUser, modifiedTime, iconLink, owners, folderColorRgb, shared, webViewLink, webContentLink), nextPageToken'
};

// リクエストオブジェクトの生成
var objReq = googleDriveClient.drive.files.list(objParam);
objReq.execute(function (resp) {
    resolve(resp);
});
```

**Google Driveのマイフォルダ配下のファイルおよびフォルダ一覧を取得する方法がわかりませんでしたが、もしご存じの方がいればご教示頂けると助かりますm(__)m**

### ファイル取得時の絞り込み

qプロパティにクエリを追加することで取得条件を追加できます。
クエリで追加可能なパラメータは[こちら](https://developers.google.com/drive/v3/web/search-parameters)になります。

```
q: 'trashed=false'
```

## フォルダ配下のファイル一覧の取得

ファイル取得する時のクエリに「フォルダID in parents」を追加で指定することで、
特定のフォルダ配下のファイル一覧を取得することができます。


```
if (strParentId) {
  objParam.q += ' and ' + '"' + strParentId + '" in parents';
}
```

## 次ページのファイル取得

ファイル一覧の取得で得られた```pageToken```をリクエストパラメータに追加することで次ページのファイル一覧が追加取得されます。

```
if (strPageToken) {
  objParam.pageToken = strPageToken;
}
```

## ファイルのプレビュー

レスポンスで得られたファイル、フォルダのメタ情報のwebViewLink（v2はalternateLink）項目にアクセスするといつものGoogle driveのプレビューが表示されます

## ファイルのダウンロード

同じレスポンスで得られたメタ情報のwebContentLink項目にアクセスするとファイルのダウンロードが開始されます。

## リビジョン別のファイルの取得

Google Driveの版管理で保存されているファイルを取得することができます。  
上記のサンプルコードにはありませんが、revisionsメソッドを利用した簡単なサンプルコード（V2版）です。  
V3のリファレンスは[こちら](https://developers.google.com/drive/v3/reference/revisions/get)  
V2のリファレンスは[こちら](https://developers.google.com/drive/v2/reference/revisions/get)  

リクエストパラメータにfieldsプロパティを追加することで、上記同様に取得するメタ情報の項目を指定することができます。  
V3のレスポンス値は[こちら](https://developers.google.com/drive/v3/reference/revisions#resource)になります。  
リファレンスにもありますようにV3ではファイル取得用のURIがメタ情報で取得することができないので、ファイルそのものを取得したい場合はV2を利用してください。  
（V3でのリビジョン別のファイル取得の方法がお分かりの場合は教えて頂ける幸いです）

V2ではfieldsに「downloadUrl」を含めることでファイルのダウンロードリンクを取得することができます。

```
// API V2

var objParam = {
  fileId: 'ファイルID',
  revisionId: 'リビジョンID',
  fields: 'id, mimeType, fileSize, downloadUrl'
}

googleDriveClient.drive.revisions.get(objParam).then(function(objRevision) {
  // 指定したファイルの特定のリビジョンのメタ情報
  console.log(objRevision);
});
```
