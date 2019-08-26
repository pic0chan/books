- 全文検索:
  - 単語単位だよ:
    - 正規表現とかは原則お門違いってことだね！
    - 何を単語の区切りにするのかは Analyzer 次第
  - 単語毎に転置インデックスへ格納:
    - 単語,DocumentID のセット

- 用語
  - Document: RDBでいうレコード
  - Field: Document 内の key, value のセット
  - Index: Document を保存. (転置インデックスする範囲かな?)
  - Document Type: どんなFieldが入るべきかデータ構造を定義する箱. 前は 1 index に N Document Typeだったけど今は 1:1.
  - Mapping: Document Type に対応づけるデータ構造を具体的に定義したもの
  - Node: ElasticSearch が動作する各サーバ. 1 つの JVM インスタンスのことが本質. つまり 1 ホストに 2 JVM 動かし 2 node といっても良い
  - Cluster: ElasticSearch を複数ノードで協調して動作する範囲
  - Shard: index を分散配置. Cluster 内で分散配置可能.
  - Replica: 各shard を複製したもの.

- Cluster
  - Master (Master-eligible) ノード
    - Master ノードになりうるホスト
    - 1 cluster に 1 台以上存在できる
```
node.master: true|false で設定
```
  - Data ノード
    - データを格納するホスト
```
node.data: true|false で設定
```
  - Ingest ノード
    - ElasticSearch5.0 から登場
    - 従来のLogstashでやるpipelineだよ
    - 今のところ俺わからん
  - Coordinating ノード
    - scatter フェーズと gather フェーズのみを実施するノード
```
node.master: false
node.data: false
node.ingest: false
```

- 処理流れ
  1: クエリ来訪
  2: scatter フェーズ = クエリ内容に対応する1つ以上のシャードへルーティング
  3: gather フェーズ = 各ノードからレスポンスを集約してクライアントへ応答

- スプリットブレイン:
  - ネットワーク断が発生したときにMasterが2台以上できて誤って書き込みはじめちゃうやばい状態
  - 防ぐためにMaster-eligibleが 過半数 いないとMasterが存在できない
    - クラスタにMasterが存在するには N/2 + 1  いる方
      - 障害ホスト数観点で言えば Down * 2 + 1 分用意すればいいってことだね！
    - 過半数とは別に? minimum_master_nodes を定義できる

- shared数 / replica数 について
  - index作った後 shard数は変えられない
  - index作った後 replica数は変えられる
  - shard数はホストを増やす計画があればoverallocationもあり得る
  - replica数は後から考えるでも大丈夫
  
- Cluster組んでみたYO
  - VM を3台用意
  - docker で elasticsearch を各 VM で起動
    - cluster 組めなかった... NAT が原因?
  - rpm で elasticsearch を各 VM で起動
    - cluster 簡単に組めた
    - 落としても大丈夫!!
  - ミスりそうポイント
    - seed は ip address or DNS名 書いておく
    - initial_master_nodes には node name を指定する
      - デフォルトは hostname

- ステミング(stem): 語形変化をそろえて、同一の表現に。dogs/dog->dog, 食べる/食べた->食べ 
- 正規化: 大文字小文字をそろえる。カタカナひらがなをそろえるなど。
- ストップワード: 単語自体に意味をもたないものを除去。the, of とか助詞とか。

- Aggregation:
  - 検索結果の集合に対して, 分類, 集計をする機能
    - Metrics: 分類されたグループに対して最小, 最大, 平均など統計値を出すもの
    - Buckets: mysqlでいうgroup by的なやつ. 一番よくつかわれるのが term range あたり
    - Pipeline: bashのpipe的な.処理を次の処理に渡せる
    - Matrix: 廃止
```json
"aggs": {
  "<agg_name>": {
    "<agg_type>": {
      "<agg_body>"
    }
  }
}
```
```json
"aggs": {
  "avg_salary_title": {
    "avg": {
      "field": "salary"
    }
  }
}
```

- Buckets


