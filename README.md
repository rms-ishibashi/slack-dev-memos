Slack上のメッセージを取得したいときのTipsやメモ。
ここではReal Time じゃない方のAPIについて。

jsonのフォーマットが統一されておらず参照しづらいので、どういうフォーマットになっていいるのか調査してみました。		

# 検証条件
自社Slack randomチャンネル
2014年4月〜2015年9月の期間中にポストされたメッセージを見てみた。

# メッセージのフォーマットについて
最低限、以下の3つの情報はないと話にならないと思われる。

- 時刻
- ユーザ名(またはID)
- POSTしたテキスト

基本的には

```json
{
	"ts": timestamp(float),
	"user": user_id(string),
	"text": text(string),
	"type": type(strin)
}
```

こういうフォーマットになっている。あとは _subtype_ 属性があったりなかったり。
_type_ 属性は _"message"_ 以外ありませんでした。

1. 時刻 _ts_ と テキスト _text_ は必ずある
2. ユーザは参照できる属性名が統一されていない

# ユーザの参照方法について
