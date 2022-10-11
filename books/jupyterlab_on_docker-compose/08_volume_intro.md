---
title: Docker Composeとローカルの話前編 - ボリューム（バインドマウント）を理解しよう
---

# ボリュームとは
この回では，Dockerにおけるボリュームについて解説していきます．
公式ドキュメントによれば，コンテナ内で生成されるデータを永続的に保持する目的で利用される仕組みです．
中々面倒なので詳しくは公式ドキュメントを読んでくれって感じですが，このBookの中で使う必要最低限の知識を確認します．

# ボリュームとバインドマウント
結論から言うと以下の三点に収まります．
- 今回主に使うのは厳密にはボリュームではなく，バインドマウントという機能になります．
- バインドマウントが「ホストマシン上のファイルシステム」に保存されるのに対して，ボリュームは文字通りデータの永続化のためにDockerが管理している領域です．
- 特にバインドマウントに関して，データの永続化というとわかりにくいですが，結果だけ見れば「コンテナとホストのファイル共有」という表現の方がわかりやすいかとおもいます．

# Docker Composeで用いる際の記法
今回使う方法は至ってシンプルです．以下のように，ローカル側のファイルパスとリモート側のファイルパスを`:`で囲むことで表現します．
```yaml
volumes:
  - ./path/to/local/dir:/path/to/container/dir
```
ローカル側のファイルパスについて，docker-compose.ymlから見た相対パスを指定するときは`.`から始めてください．