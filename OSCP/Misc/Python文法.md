すこし理解に時間がかかった文法をまとめる。
公開エクスプロイトのほとんどがPythonであるため、Pythonの理解を深める必要がある。

---

# Mainについて

## main関数の定義

- すべての関数を定義してから、mainでプログラムを実行する
- そのため、mainの定義は一番最後になっている
- ↓具体例
```python
import requests
import sys

def usage():
	if len(sys.argv) != 4:
		print("Usage: python3 exploit.py [URL]")
		sys.exit(0)

def copy_cut(url, session_cookie, file_name):
	headers = {'Cookie': session_cookie,
	'Content-Type': 'application/x-www-form-urlencoded'}
	url_copy = "%s/filemanager/ajax_calls.php?action=copy_cut" % (url)
	r = requests.post(
	url_copy, data="sub_action=copy&path=../../../../../../.."+file_name,headers=headers)
	return r.status_code

def main(): # ← 最後に定義
	usage()
	url = sys.argv[1]
...

if __name__ == "__main__":
	main()
```

## `__name__` とは？

Pythonでは、実行されているモジュールには `__name__` という特殊変数が自動で入る
- 直接実行された場合 → `"__main__"`    
- 他のファイルから `import` された場合  → `__name__ == ファイル名（拡張子なし）`

### `__name__`の具体例

1. test_module.pyを定義
```python
print("このファイルの __name__:", __name__)

def hello():
    print("Hello!")

if __name__ == "__main__":
    print("直接実行されました")
```

2. 直接実行
```python
python3 test_module.py
# このファイルの __name__: __main__
# Hello!
# 直接実行されました
```

3. main.pyを定義
```python
import test_module # test_module.pyをモジュールとしてインポート

print("main.pyの __name__:", __name__)
```

4. main.pyを実行して、test_module.pyをモジュールとして実行
```python
python3 main.py
# このファイルの __name__: test_module
# Hello!
# main.pyの __name__ : __test_module__
```

---

# 文字列操作

## 文字列の置き換え

```python
print 'My name is %s.' % ('Ikeda')
# My name is Ikeda.
```
```python
# exploit.py http://examle
url = sys.argv[1]
url_expliot= "%s/../../../etc/passwd" % (url)
print url_exploit
# http://example/../../etc/passwd
```

