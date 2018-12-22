# NOTE

Fork元のコメントを日本語訳しました。

# brat docker

これは[brat](http://brat.nlplab.org/)サーバをデプロイするdockerコンテナです。


### インストール

コンテナにアノテーションデータと設定を渡すために２つのボリュームを作成する必要があります。
以下の通り、それぞれに名前をつけてボリュームを作成します。

```bash
$ docker volume create --name brat-data
$ docker volume create --name brat-cfg
```

`brat-data`ボリュームはアノテーションデータにリンクさせ、`brat-cfg`ボリュームは`users.json`ファイルを入れてください。
複数ユーザが追加するためには`users.json`に以下のようにユーザ名とパスワードを列挙してください:

```javascript
{
    "user1": "password",
    "user2": "password",
    ...
}
```

これらのディレクトリ内のデータはコンテナが停止・削除されても永続的に残ります。
その後、別のbratコンテナを起動し、同じデータを見ることができます。
`docker < 1.9.0`を使用している場合はnamed volumeを使用出来ないので、代わりにdata-only containerと`--volumes-from`を使う必要があることに気をつけてください。

コンテナ内からデータの追加、ユーザの編集をすることもできます。コンテナ内に直接データを追加するには以下のように行ってください:
``` bash
$ docker run --name=brat-tmp -it -v brat-data:/bratdata cassj/brat /bin/bash
$ cd /bratdata
$ wget http://my.url/some-dataset.tgz
$ tar -xvzf some-dataset.tgz
$ exit  
$ docker rm brat-tmp
```

データがホストマシーンにある場合は、dockerがどこにnamed volumeを保存しているかチェックし、: 

```bash
$ docker volume inspect brat-data 
```
そこに直接データをコピーすることができます。


### 起動

初めて起動するときはコンテナ起動時にユーザネーム、パスワード、emailを環境変数として与えてください。このユーザが編集権限を持ちます。
```bash
$ docker run --name=brat -d -p 80:80 -v brat-data:/bratdata -v brat-cfg:/bratcfg -e BRAT_USERNAME=brat -e BRAT_PASSWORD=brat -e BRAT_EMAIL=brat@example.com cassj/brat
```
