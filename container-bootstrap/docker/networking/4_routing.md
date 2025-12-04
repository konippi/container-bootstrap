# ルーティング

コンテナから外部ネットワークに接続できるよう IP マスカレードが行われるようルーティング設定されている。

## ルーティング設定の確認

1. docker0 の IP を確認する (デフォルト: 172.17.0.1/16)

```shell
$ ip addr show | grep docker0

3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
```

2. iptable を確認する

```shell
$ sudo iptables-save

-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

- 送信元: `172.17.0.0/16` (コンテナ)
- 送信先: docker0 以外のインタフェース
- 処理: MASQUERADE (送信元 IP をホストの IP に変換)

上記を踏まえると以下の図でルーティングされていることがわかる。

<img width="400" alt="docker0" src="../../../asset/img/docker-network-routing.png" />
