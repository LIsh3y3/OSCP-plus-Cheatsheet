- **注意**：正確性はIntruderより低い
	- なぜかどうしても送れないpayloadなどがある
###### 参考：

- [概要（Qiita）](https://qiita.com/shinkbr/items/985dfb6e28dc23aa62d5)
- [Youtube: 2FA broken logic](https://www.youtube.com/watch?v=iYbM89TuZkw)
- [公式ドキュメント](https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack)

## 基本機能

- basic.pyを元に解説
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )

    for word in open('/usr/share/dict/words'):
        engine.queue(target.req, word.rstrip())


def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if req.status != 404:
        table.add(req)

```
###### concurrentConnections

- 同時接続数
- 10~30のレンジで素早さが増す
###### requestsPerConnection

- 接続ごとのリクエスト数。
- 100~200のレンジ。RetriesやFailsが大きい場合は減らす。
- ⚠️正確な挙動を観察したいときはconcurrentConnectionsとrequestsPerConnectionを1に設定
###### pipeline

- False = 一度に１つのRequest
- True = 一度に複数のRequest. 早い。サーバもパイプライン処理に対応してないといけないが。

###### for word in open('{filename}')

- filenameにwordlistを指定
- 標準ではBurp Suite Academy提供のwordlist
- `engine.queue~`はwordlistの単語を一つずつ準備しているだけ

###### handleResponce

- 特定の結果にフラグを立てられる
- ⭐️`req.status`,`req.wordcount`, `req.length`, `req.response, interesting`が利用可能

- 例えばレスポンスのメッセージをフラグに設定できる
```python
if 'Invalid Password' in req.responce:
	table.add(req)
```
- ⚠️：req.lengthの方がFalisとならない。Repeaterで右下の〇〇bytesがlengthなので、これでフィルタする

---
## 応用機能

- [[⚡️File upload vuln#Turbo Intruderで複数リクエストを送る]]

- `multipleParameters.py`で複数パラメータを送信する方法が載っている

- `benchmark-h2-race.py`に[[3. 隠されたMulti-step race conditionの基本と方法論#②手がかりを探る]]のベンチマークとして使えるリクエストが載っている