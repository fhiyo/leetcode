# 929. Unique Email Addresses

## 1st

### ①

正規表現が最初に思いついた。ReDOSが怖いからPythonで正規表現を簡単に選択肢に入れない方がいいのだろうか？

答えを返す部分のlambdaは不要で、単に `map(normalize, emails)` で良かった。

所要時間: 15:35

正規表現の計算量、よく分からない。normalize一回あたりで文字列の長さをnとして考えると、
- 時間計算量: 最悪指数時間?これはバックトラッキングで指数時間かかる場合がある？無いとすると、O(n)
- 空間計算量: O(n)か?

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            g = re.match('(.*)@(.*)', email)
            local_name = g[1]
            domain_name = g[2]
            normalized_local_name = re.sub('\.|\+.*', '', local_name)
            return ''.join([normalized_local_name, '@', domain_name])

        return len(set(map(lambda e: normalize(e), emails)))
```

### ②

rsplitで分けてreplace, findで削除。discord上でlocal_partの中で@が含まれる場合について話していた記憶があったので、一応右から検索した (https://discord.com/channels/1084280443945353267/1201211204547383386/1209856265413591140 にあることを確認)。
[RFC5322](https://datatracker.ietf.org/doc/html/rfc5322#section-3.4.1)と[RFC1034](https://www.ietf.org/rfc/rfc1034.txt)を見る限り、domain側に@は多分含まない、と思う。

入力が不正で@が含まれない場合、 `local_name, domain_name = email.rsplit('@', maxsplit=1)` の行でunpackできずにエラーになる。まあいいんじゃないだろうか

所要時間: 7:00

normalize一回あたり、文字列の長さをnとして、
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_name, domain_name = email.rsplit('@', maxsplit=1)
            local_name = local_name.replace('.', '')
            plus_position = local_name.find('+')
            if plus_position != -1:
                local_name = local_name[:plus_position]
            return ''.join([local_name, '@', domain_name])

        return len(set(map(normalize, emails)))
```

### ③

local_nameを一文字ずつ進めるパターン。listと文字列を両方normalized_local_nameという同じ変数に入れているのはあまり行儀が良くないかもしれない。文字列もリストのようなものだしまあいいか、と考えた

所要時間: 5:02

normalize一回あたり、文字列の長さをnとして、
- 時間計算量: O(n)
- 空間計算量: O(n)

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_name, domain_name = email.rsplit('@', maxsplit=1)
            normalized_local_name = []
            for ch in local_name:
                if ch == '+':
                    break
                if ch == '.':
                    continue
                normalized_local_name.append(ch)
            normalized_local_name = ''.join(normalized_local_name)
            return ''.join([normalized_local_name, '@', domain_name])

        return len(set(map(normalize, emails)))
```


## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1200089668901937312/1207717581390217236
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1209166861821026356
    RFCへのリンクが書いてある。
- https://discord.com/channels/1084280443945353267/1200089668901937312/1210065979300651048

splitしないでindexを進めるパターンがあったのでそれもやる。

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> Generator[str, None, None]:
            local_part = True
            ignored_local_part = False
            for ch in email:
                if not local_part:
                    yield ch
                    continue
                if ch == '@':
                    local_part = False
                    ignored_local_part = False
                    yield ch
                    continue
                if ignored_local_part or ch == '.':
                    continue
                if ch == '+':
                    ignored_local_part = True
                    continue
                yield ch

        return len(set(map(lambda e: ''.join(normalize(e)), emails)))
```

yieldする形にした。2回書いて整ってきたものを載せている。


- https://discord.com/channels/1084280443945353267/1200089668901937312/1210619083385479258
  文字列追加 += が最適化される話と、環境に対して頑健か、という話。

## 3rd

joinしているところはf-stringで書いてもよかったかもと後で思った。

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_part, domain = email.rsplit('@', maxsplit=1)
            local_part = re.sub('\.|\+.*', '', local_part)
            return ''.join([local_part, '@', domain])

        return len(set(map(normalize, emails)))
```

```py
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_part, domain = email.rsplit('@', maxsplit=1)
            local_part = local_part.replace('.', '')
            plus_position = local_part.find('+')
            if plus_position != -1:
                local_part = local_part[:plus_position]
            return ''.join([local_part, '@', domain])

        return len(set(map(normalize, emails)))
```
