2015, 10/1 Update

Slack上のメッセージを取得したいときのTipsやメモ。

このメモでやったこと -> jsonのフォーマットが統一されておらず参照しづらいので、どういうフォーマットになっているのか調査してみた

#️ 要件
最低限、以下の3つの情報はないと話にならないと思われる。

- 時刻
- ユーザ名(ID)
- POSTしたテキスト

全ポストからこの3つの情報を取得したい、っていうのが今回の要件。

# 結論
時刻とテキストについては、

```python
data = json.load(open("posts.json"))
for msg in data:
	print msg.get("ts", "")
	print msg.get("text", "")
```

のような感じでOK。

_id_ の取得については、以下の条件で取得できないものはスルー。
検証条件の下では、これに該当するポストは全体の0.2[%]だったので無視して良いと結論づけます。

```python
def get_user_id(m):
    if m.has_key("user"): return m["user"]
    if m.has_key("bot_id"): return m["bot_id"]
    return None

for msg in data:
	uid = get_user_id(msg)
	if uid is None:
		continue
	# do something ...
```

# 検証条件
データは自社の _random_ チャンネルから。
取得期間はおよそ1年半分ほど。

# メッセージのフォーマットについて
基本的には

```json
{
	"ts": "timestamp(float)",
	"user": "user_id(string)",
	"text": "text(string)",
	"type": "type(string)"
}
```

こういうフォーマットになっている。あと、これらの属性に加えて _subtype_ 属性があったりなかったり。

※ _type_ 属性については _"message"_ 以外の値がなかった。

で、１段目の要素を見ていると次のような結果が出た。

1. 時刻 _ts_ と テキスト _text_ は必ずある
2. ユーザを参照する属性名が統一されていない。
3. トップレベルからはユーザ情報を参照できない場合さえある


-----


残りは雑多なメモ。
そのうち整理する予定。。

# ユーザIDの参照方法について
基本的には _"user"_ を参照すればIDを取得できる。

_user_ での参照ができなかったポストは以下のパターン:

1. botによるポスト
2. _subtype_ が _"file_share"_ の場合

botの場合でも _"user"_ で参照できる場合はあるので、botによるポストが全て _user_ 属性を持たないわけではない模様。
仕様の変更によるものか、それとも何がしかの条件でそうなってるのか。。。

## _subtype_ が _"bot_message"_ の場合

## _subtype_ が _"file_share"_ の場合
_subtype_ が _"file_share"_ のときは画像のアップロードが行われた場合が該当。
ただし、アップロードが取り消された場合には次のようなjsonが返るのでユーザを参照することは不可能。

```json
 {u'file': None,
  u'subtype': u'file_share',
  u'text': u'A file was shared',
  u'ts': 1423224740.000374,
  u'type': u'message',
  u'upload': False}
```	

この場合はおとなしくスルーするぐらいしか手がないと思われる。それ以外であれば _username_ 属性を参照すればユーザのIDを見ることができる。


# MEMO
_comment_ 属性がトップレベルに存在する場合は _comment_ の中を見れば最低限の情報は参照できる。例えばこんな感じ:

```json
{u'comment': u'',
  u'created': 1438946160,
  u'id': u'zzzzzzzzz',
  u'timestamp': 1438946160,
  u'user': u'Uxxxxxxxx'}
```	

この場合は

- 時刻: message["comment"]["timestamp"]
- ユーザ: message["comment"]["user"]
- テキスト: message["comment"]["comment"]

こんな感じで参照が可能。


```python
data = json.load(open("posts.json"))
for m in data:
	if x.has_key("comment"):
		pass // comment属性以下から情報を取得
	if 