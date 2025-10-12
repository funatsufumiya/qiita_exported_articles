DNSサーバを自分で設置したり、設定をいじっていると、サーバを明示的に指定して`nslookup`や`dig`したいことがあります。

いつもやり方を忘れるのでリストにしました。

## DNSサーバを指定して、解決後のIPアドレスを取得

```bash
nslookup google.com 8.8.8.8
# nslookup [調べたいドメイン名] [DNSサーバのアドレス]

# --- 実行結果 (`nslookup google.com 8.8.8.8`) ---
# Server:		8.8.8.8
# Address:	8.8.8.8#53
#
# Non-authoritative answer:
# Name:	google.com
# Address: 172.217.26.46
```

nslookupをインタラクティブモードで使う場合は、以下のようにハイフンを入れます。

```bash
nslookup - 8.8.8.8
# > google.com (解決したいドメイン名を入れてエンターを押すと、インタラクティブに結果が返ってくる)
## Server:		8.8.8.8
## Address:	8.8.8.8#53
##
## Non-authoritative answer:
## Name:	google.com
## Address: 172.217.25.78
# > ...
```

## digコマンドで同じことをするには

```bash
dig @8.8.8.8 google.com
# dig @[DNSサーバのアドレス] [調べたいドメイン名]

# --- 実行結果 (`dig @8.8.8.8 google.com`) ---
# ; <<>> DiG 9.10.6 <<>> @8.8.8.8 google.com
# ; (1 server found)
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23813
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
#
# ;; OPT PSEUDOSECTION:
# ; EDNS: version: 0, flags:; udp: 512
# ;; QUESTION SECTION:
# ;google.com.			IN	A
#
# ;; ANSWER SECTION:
# google.com.		69	IN	A	172.217.26.46
#
# ;; Query time: 36 msec
# ;; SERVER: 8.8.8.8#53(8.8.8.8)
# ;; WHEN: Thu Oct 03 12:17:25 JST 2019
# ;; MSG SIZE  rcvd: 55
```

nslookupよりも詳細なデータが表示されます。

### おまけ: どのDNSサーバから返ってきた情報なのかを得る方法

ANSWER SECTION が返答に追加されて、ネームサーバの詳細がわかるようになります。

```bash
dig google.com ns

# --- 実行結果 (`dig google.com ns`) ---
# ; <<>> DiG 9.10.6 <<>> google.com ns
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33847
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1
#
# ;; OPT PSEUDOSECTION:
# ; EDNS: version: 0, flags:; udp: 512
# ;; QUESTION SECTION:
# ;google.com.			IN	NS
#
# ;; ANSWER SECTION:
# google.com.		21599	IN	NS	ns3.google.com.
# google.com.		21599	IN	NS	ns1.google.com.
# google.com.		21599	IN	NS	ns4.google.com.
# google.com.		21599	IN	NS	ns2.google.com.
#
# ;; Query time: 102 msec
# ;; SERVER: 8.8.8.8#53(8.8.8.8)
# ;; WHEN: Thu Oct 03 12:12:08 JST 2019
# ;; MSG SIZE  rcvd: 111
```

ネームサーバ指定と組み合わせることも可能です。

```bash
dig @8.8.8.8 google.com ns

# --- 実行結果 (`dig @8.8.8.8 google.com ns`) ---
# ; <<>> DiG 9.10.6 <<>> @8.8.8.8 google.com ns
# ; (1 server found)
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26086
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1
#
# ;; OPT PSEUDOSECTION:
# ; EDNS: version: 0, flags:; udp: 512
# ;; QUESTION SECTION:
# ;google.com.			IN	NS
#
# ;; ANSWER SECTION:
# google.com.		21599	IN	NS	ns3.google.com.
# google.com.		21599	IN	NS	ns1.google.com.
# google.com.		21599	IN	NS	ns2.google.com.
# google.com.		21599	IN	NS	ns4.google.com.
#
# ;; Query time: 76 msec
# ;; SERVER: 8.8.8.8#53(8.8.8.8)
# ;; WHEN: Thu Oct 03 12:13:16 JST 2019
# ;; MSG SIZE  rcvd: 111
```
