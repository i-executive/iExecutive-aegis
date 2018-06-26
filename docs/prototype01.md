# prototype ver.1

プロトタイプ01

# 目的

smart contractで文書のハッシュ情報を登録できることを確認する。

もっとも、ベーシックな方法でインプリする。ERC20のイメージで。

ただ、この形だと更新時のコストが高そうだが、実際にはそうでないという部分を確認する意味でも。
- mappingオブジェクトで全体のmappingを管理するアーキテクチャでデザインされている。
- この場合には一般的にはコストが高そうだがそうでもないのか？


# 構成

- EOA
    * 文書管理
    * proto1では、文書をEOAとする。


- SmartContract
    * 文書登録のAPIの提供
    * 全文書をmappingで管理
        * mapping配下に文書情報indexを管理する。

- transaction
    * 文書情報を登録

- 文書情報
    * uri 
    * hash
    * owner
    * status

# Actors

- 人 : person
- 文書作成者 : owner
- 文書承認者 : approver

- 文書 : document
- 文書セット ; document-set
- 文書管理レポジトリ : repository

- ワークフロー : workflow
- ワークフローシステム : workflow system


文書作成者によって作成された文書グループに含まれる各文書は文書管理レポジトリに保管され、
決済処理を定めたワークフローに従い、文書承認者がワークフローシステムを使用して決済する。

本プロトタイプでは、ワークフローシステムから利用されるサブシステムとして文書のサインを
blockchain上の台帳で管理するシステムです。

文書情報はURIによってユニークに定められるドキュメントセットの情報であり、以下の情報を
有するものと定義する。

- 文書URI
    - 文書管理レポジトリで管理される文書セットを示すユニークな識別子
- 文書セット ハッシュ
    - 文書セットのスナップショットを示すハッシュ値
- owner
- status
- last approvers
- approvers

人の情報はblockchain上に管理される。署名に利用したpubkey(hash)に紐付け、IDおよび参照先システムが登録される。
オプショナルで個人情報も登録できるが、外部のシステム参照で個人情報は参照される。

文書情報更新においては、コインと同様に


- coinと同じ処理で考えた場合にロールバック的な処理については、人に依存してしまいかねない。システムで処理できるような仕組みを検討する？
    - システム処理を中心に考え、承認手続きについてログを残す部分が本人性が偽証できないようにすることが大事。
    - つまりドキュメントの管理はシステムに常にあると考える。つまり文書を人につけないこととする。

- 文書にサインするというイメージで考える。
    - PGPのキーリング？のイメージをblockchain上に文書IDに対して行うと考える。
    - 文書の最新世代に対してのサインと文書IDに対するサインをコンセプト上分離するべきか？
        - 承認中のコメントに対して修正したドキュメントセットに対して過去の承認者の再承認フローをどう考えるか？
            - ワークフローの定義で制御するものと考え、本システム要件とはしない。
            - 再承認はオプショナルでよいかな。
            - 関係者リストと最終承認者リストを分ければ良いか

- タイムスタンプの考え方
    - コントラクト自体に厳密なタイムスタンプを生成できない。
    - 外部システムでタイムスタンプを設定してログとするか？
        - タイムスタンプは本来は偽証できないように管理されたシステムで付与するような仕組みが求められる。
        - コントラクトをラップするようなタイムスタンプシステムを構成することで、回避する？


## architectures

- contract base modules

    - document signing system
        - register document
            - owner's pubkey, sign
            - document uri
            - document revision
            - document hash

        - sign document
            - signer's publkey, sign
            - new status
            - document id
            - document hash

        - update document
            - owner's pubkey, sign
            - document id
            - update document revision
            - update document hash



    - person directory
        - register person
            - person manager uri
            - person managed id
            - optional info
            - person's pubkey, sign
            - return: person id(hash)
        - update person
            ...
        

- external modules

    - workflow system
    - repository



## primitive function

秘密鍵と公開鍵による組み合わせで本人証明を行う仕組みの実装

つまり、walletの処理を実装する。

1. 本人性の確認は公開鍵と対応した署名で検証を行う。
    - ecrecoverを使用して行う。

    1. メッセージそのもののハッシュ(sha-3)の生成
    2. パラメタの署名
    3. 上記から署名者と同一かを確認する。
    4. 有効な署名者として、継続処理を実行する。


    以下の関数の組み合わせで実現する必要がある(solidityで用意されている暗号関数は以下しかない。)。代表的なものはハッシュを計算するもの。


    ecrecoverを使用することで、元メッセージのハッシュとアカウントがサインした(web3.eth.sign()で生成した)signature(v,r,s)から
    アドレス(pubkeyのsha-3の後ろ20バイト)を求めることができる。

    メッセージのハッシュ作成は以下のコードを参考に.ハッシュする文字列の前に'\x19Ethereum Signed Message:\n'とハッシュ文字列長を
    必ず設定する必要があります。

    しなくても生メッセージをそのままHashしても良いかもしれない。（未検証)


    ``` js
        const message = 'Message to sign here.'
        const hexMessage = '0x' + Buffer.from(message).toString('hex');                    
        const unlockedAccount = accounts[0]                    
        const hashedMessage = web3.sha3('\x19Ethereum Signed Message:\n' + message.length + message)

        var signature = await web3.eth.sign(unlockedAccount, hexMessage)
        r = signature.substr(0, 66)
        s = '0x' + signature.substr(66, 64)
        v = '0x' + signature.substr(130, 2)
        const recoveredAddress = await this.contract.ecrecover2(hashedMessage, v, r, s)
    ```


    - Cryptographic Functions
        * sha3(...) returns (bytes32): compute the SHA3 hash of the (tightly packed) arguments
        * sha256(...) returns (bytes32): compute the SHA256 hash of the (tightly packed) arguments
        * ripemd160(...) returns (bytes20): compute RIPEMD of 256 the (tightly packed) arguments
        * ecrecover(bytes32, byte, bytes32, bytes32) returns (address): recover public key from elliptic curve signature

## person directory

- プロトの実装では、EOAに人をマッピングする。
- 外部システムにEOAのアカウントと個人情報を紐づけておく。
- 紐づけられた情報を承認済みアカウントとしてperson directoryに登録する。
- 登録のパーソンIDとして新規に生成するかEOAを使用するかは検討が必要。
- あまり間接参照を多くするとコストが高くなるので。。。

### functions

#### register person

- parameters
    * authority uri:
    * authority person id:
    * optional : 
    * signature

- flow
    1. 端末から authority.getPerson(person-id)でユーザ情報を取得
    2. ユーザ情報をweb3.eth.sign()でsignatureを生成
    3. authority.registerLedger(person-id, signature)で台帳登録を要請
    4. authorityは person directory contractにユーザ情報(含むEOA)およびsignatureをパラメタにregister personを実行
    5. registerPersonでは、signatureとユーザ情報からecrecoverを呼び出しEOAを検証し、成功したらEOAをキーにユーザ情報を登録する。

#### validate person : 人の検証

- parameters
    * EOA

- flow
    1. EOAがmappingに存在しているか確認する。
    2. 存在していればOK



## document signing system

### function

#### register document

- parameters
    * repository uri
    * document revision
    * document hash
    * owner 
    * owner's signature

- flow
    1. ドキュメントレポジトリ上のドキュメントセットのURIを取得
    2. ドキュメントセットのリビジョン情報(gitのcommit id相当)
    3. ドキュメントセットのハッシュ値の計算
    4. 上記情報とドキュメントのオーナーのEOAを含めたデータにsign(web3.eth.sign)
    5. document signing system contractのregisterDocumentをcall(というかsendTransaction)
    6. contractではsignとドキュメント情報のhashからEOAを検証
    7. ドキュメントIDを生成（何らかのロジックで...>> signatureでいいかな) : トランザクションは確か復帰情報が返せないので、呼び出し元でも把握できる値である必要がありそう
    
    8. ドキュメントIDをキーにmappingにドキュメント情報を登録する


# 機能

## コントラクト


## ローカルアプリ

- ドキュメントの秘密鍵を持ち、ドキュメントのステータスを更新可能なエンドポイント


# 確認項目

どの情報を秘密鍵でサインするのか？

いわゆるwalletで非公開情報的な部分をどこにするのか？

- 公開鍵＆署名(秘密鍵で署名)を送って真贋チェックしている。
- 仮想通貨では秘密情報はなし、秘密鍵の所有者チェックを行ってトランザクションの発行をしているにすぎない。
- アプリ側に秘密鍵を持たせることで、ネットワーク上には秘密鍵は展開しない。ローカルのアプリで閉じる。
- 登場人物(実行空間)として、ローカルアプリ、スマートコントラクトがある。




## 文書トランザクションを登録

### synopsis

- registerDocument
- updateDocument

## 文書トランザクションを取得

- getDocument( docId)