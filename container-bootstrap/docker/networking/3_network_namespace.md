# Network Namespace

> [!IMPORTANT]
> 最初に `colima ssh` で Colima VM (Linux) に SSH で接続することを忘れない

Linux カーネルが提供する Namespace の一種で、ネットワークスタックを分離・仮想化する機能。

<h4>特徴</h4>

- **分離性**: 異なるnamespace間でネットワークリソースが完全に分離
- **独立したネットワークスタック**: 以下のネットワークリソースが独立
  - ネットワークインタフェース
  - IP アドレス
  - ルーティングテーブル
  - etc...

## ホストとコンテナにおけるインタフェースの違い

1. コンテナを実行

```shell
# コンテナを立ち上げる
$ docker run --rm -d --name nginx nginx

# ホストのインタフェースを確認する
$ ip link show | grep veth

23: vethe0736e0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
```

2. コンテナ内からインタフェースを確認

```shell
# コンテナに入る
$ docker exec -it nginx /bin/sh

# ip コマンドを使えるようにする
$ apt-get update && apt-get install iproute2 -y

# コンテナインタフェースを確認する
$ ip link show | grep eth0

2: eth0@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default

# コンテナから出る
$ exit
```

ホストとコンテナにおけるインタフェースを比較すると...

- コンテナ側の veth はホストとのインタフェースなので eth0 と命名している
- `@if` を見るとお互いのインタフェースインデックスと一致している

## Network Namespace による分離

1. ホストの Network Namespace を確認

```shell
# Network Namespace の inode を確認する
$ ls -la /proc/self/ns/net

lrwxrwxrwx 1 konippi konippi 0 Dec  4 17:14 /proc/self/ns/net -> 'net:[4026531840]'
```

2. コンテナの Network Namespace を確認

```shell
# コンテナに入る
$ docker exec -it nginx /bin/sh

# Network Namespace の inode を確認する
$ ls -la /proc/self/ns/net

lrwxrwxrwx 1 root root 0 Dec  4 08:14 /proc/self/ns/net -> 'net:[4026532384]'

# コンテナから出る
$ exit
```

それぞれの inode 番号確認すると違うことがわかる。

- ホスト: 4026531840
- コンテナ: 4026532384

## クリーンアップ

```shell
$ docker rm -f $(docker ps -q | head -n 1)
```

<div align="right"><a href="./4_routing.md">次のページ</a></div>
