# Ethereum プライベートネットワーク構築
Docker で Ethereum のプライベートネットワークを構築する。マイニングノード2台と bootnode の3台構成とする。

bootnode のイメージ作成方法は以下のページを参照

- [bootnode コマンドが入った Docker イメージを作成する - Qiita](https://qiita.com/grohiro/items/127dfc02f2a90df6fc66)

## 初期化

初回に一度だけ実行する処理。

### 作業ディレクトリの作成

```bash
$ mkdir -p storage/{private,public}
```

### bootnode の nodekey 生成

```bash
$ docker-compose run bootnode bootnode -genkey=/var/ethereum/boot.key
```

### genesis.json の作成

genesis.json を storage/geth-{private,public} にコピーする。

```json:genesis.json
{
    "nonce": "0x00006d6f7264656e",
    "difficulty": "0x200",
    "mixhash": "0x00000000000000000000000000000000000000647572616c65787365646c6578",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "timestamp": "0x00",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x",
    "gasLimit": "0x2FEFD8",
    "alloc": {
    }
}
```

### ethereum の初期化

```bash
$ docker-compose run geth-private --datadir /var/ethereum init /genesis.json
$ docker-compose run geth-public --datadir /var/ethereum init /genesis.json
```

### アカウント作成

パスワードファイル storage/geth-{private,public} に作成してからコマンドを実行する。

```
$ echo 'mypassword' > storage/geth-private/passwd
$ docker-compose run geth-private account new

$ echo 'mypassword' > storage/geth-public/passwd
$ docker-compose run geth-public account new
```

## 起動
Ethereum ネットワークを起動する。起動する前に bootnode の接続先を更新する。

### bootnode の起動と設定
IPアドレスを確定させるために bootnode のみ起動する。
docker-compose.yml の geth-{private,public} 起動パラメータに記載された bootnodes のアドレスを更新する。

```bash
$ docker-compose up -d bootnode

# IPアドレスを確認する
$ docker-compose run bootnode ifconfig 
# nodekey を確認する
$ docker-compose run bootnode bootnode -nodekey=/var/ethereum/boot.key -writeaddress
```

表示された情報を使って `enode://` で始まるアドレスを作成して docker-compose.yml を変更する。

```
enode://[表示されたnodekey]@[IPアドレス]:30301
(例)
enode://bd0a688a5ec7d4f0989b829fc331b167c01630400cb84c8d2b9ff9b3422a68562ee16c610c4ca36794a50638c063c5a36a4c1353143301d459efa5bd433956e3@172.22.0.2:30301
```

### ネットワーク起動

```
$ docker-compose up -d

# DAG生成が終わるまで待つ
INFO [11-16|12:47:23.428] Generating DAG in progress               epoch=1 percentage=64 elapsed=2m47.877s
```


