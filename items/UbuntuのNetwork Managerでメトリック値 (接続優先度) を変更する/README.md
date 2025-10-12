ネットワークのメトリック値 (接続優先度、低いほど優先) を適切に設定することで、複数のネットワークに同時接続しても安定して使用できるようになる（例えばWi-Fiを常に有線LANより優先するなど）。

`ifmetric` コマンドでメトリック値を変更しても、一時的にしか反映されないので、Network Managerを使っている場合は `nmcli`から変更する。

```bash
# 確認
sudo nmcli connection show # 接続一覧
sudo nmcli connection show "Wired connection 1" # 設定値一覧
sudo nmcli connection show "Wired connection 1" | grep metric # メトリック値確認

# 設定変更
sudo nmcli connection modify "Wired connection 1" ipv4.route-metric 100
sudo nmcli connection modify "Wired connection 2" ipv4.route-metric 200

# 適用
sudo nmcli connection up "Wired connection 1"
sudo nmcli connection up "Wired connection 2"

# メトリック値を ip route で確認
ip route
```

なお、ipコマンドやifconfigコマンドが入っていない場合は、`apt install dnsutils` からインストールできる。

## 2022/05/16 追記

最近 Ubuntu 22.04 を使ったところ、上記設定だけではDNSの解決がうまく行かず、以下のように`ipv4.dns-priority`の設定も必要だった。

```bash
# 設定変更
sudo nmcli connection modify "Wired connection 1" ipv4.dns-priority 100
sudo nmcli connection modify "Wired connection 2" ipv4.dns-priority 200

# 適用
sudo nmcli connection up "Wired connection 1"
sudo nmcli connection up "Wired connection 2"
```
