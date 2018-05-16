# aegis@iExecutive

改ざん防止文書管理システム

## GOAL

- ドキュメントの更新・承認履歴をブロックチェーン上で管理
- 公開ドキュメントにハッシュと同様にブロックチェーン上のトランザクションを紐付け、ドキュメントに権威づけする。
- 管理ブロックチェーンの見える化

## subsystems

aegisは以下のサブシステムから構成される。ただし、全て新規に作るのではなく、既存のOSSなどで要件に合致
するものがあれば最大限活用する。

- history ledger : 更新・承認履歴台帳
- workflow : ドキュメント処理
- document repository : 文書保管
- document manager : 見える化

## TODO

- 開発者向けサイト作成 GitHub Pages
- www.dev.iexecutive.orgのDNS作成
- aegisの基本アーキテクチャの作成
- ブロックチェーンの試作
  - ブロックチェーンのデータ構造とaegisのデータ構造のマッピングドラフトを作成
- private blockchainの動作環境の構築
　　　- azure VMに構築
   
