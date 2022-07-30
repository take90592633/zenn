---
title: "【MyBatis】「要素のコンテンツは、整形式の文字データまたはマークアップで構成されている必要があります」の解決方法"
emoji: "✏️"
type: "tech"
topics:
  - "mybatis"
published: true
published_at: "2022-07-30 21:44"
---

## エラー内容
```
Cause: org.xml.sax.SAXParseException; lineNumber: XX; columnNumber: XX; 要素のコンテンツは、整形式の文字データまたはマークアップで構成されている必要があります。 
```
## 解決方法
不等号を使っている箇所を`<![CDATA[ ... ]]>`で囲む
:::message
たまにSQL全部を`<![CDATA[ ... ]]>`で囲んでしまえばOKと言うような記事も見かけますが、以下サイトの通り最低限の箇所を囲むのが良さそうです。
> <![CDATA[...]]>の中では動的SQLが有効になりません。<![CDATA[...]]>は必要最低限の箇所に利用するようにしてください。

https://qiita.com/5zm/items/0864d6641c65f976d415#19-%E3%81%AF%E3%82%A8%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%97%E5%87%A6%E7%90%86%E3%82%92%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8
:::

## サンプルソース
Kotlinでのサンプル。
```
@Mapper
interface userMapper{
    @Select(
        """<script>
	    SELECT 
	        name
	    FROM 
	        user
	    WHERE
	        id IN
		<foreach item="id" collection="ids" open="(" separator="," close=")" >
                    #{id}
                </foreach>
		AND delete_flg <![CDATA[ <> ]]> '0'
	</script>"""
    )
    fun findUsersByIds(ids: Long): List<User>
}
```

## 注意点
scriptで囲ってない時には`<![CDATA[ ... ]]>`は不要。使ってしまうと逆にエラーになるので注意。