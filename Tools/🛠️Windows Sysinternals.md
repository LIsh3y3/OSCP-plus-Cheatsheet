# Process Monitor

🔗[Process Monitor](https://learn.microsoft.com/ja-jp/sysinternals/downloads/procmon)

## Process Monitor（Procmon）とは？その用途

- プロセス、スレッド、ファイルシステム、レジストリ関連の活動に関する情報をリアルタイムで表示できる
- サービスバイナリがロードするDLLを識別することなどに使われる
	- →[Service Exploits - DLL Hijacking](../Cheatsheet/🪟Windows/💥Windows%20Privilege%20Escalation.md#Service%20Exploits%20-%20DLL%20Hijacking)

## Process Monitorの使用方法

- 使用条件として、管理者権限が必要
	- 足場のマシン上で管理者権限を持たない場合は、分析対象のバイナリをローカルマシンにコピーし、管理者権限で分析する

- メニューバーのFileter > Filter > Addより、フィルターが可能
	- 複数条件をAddしていき、最後にApplyするとフィルターが適用される（下イメージの✅）

![](../Images/Pasted%20image%2020250815125003.png)

$$Procmonでフィルターを適用したイメージ$$

---

# PsLoggedOn

🔗[PsLoggedOn](https://learn.microsoft.com/en-us/sysinternals/downloads/psloggedon)

## PsLoggedOnとは？その用途

- リモート／ローカルのマシンに誰がログオンしているかを列挙するツール

- 主に`HKEY_USERS`を参照してログオン中のユーザーの SID を取り、それをユーザー名に解決する仕組み

- さらに、`NetSessionEnum`APIを併用して共有経由のセッションも確認する
	- NetSessionEnum：サーバー上で確立されたセッションに関する情報を集める

- NetSessionEnum が使えないケースの補完手段として有用

>[!NOTE]
>- 動作するかどうかは、Remote Registry サービスに依存する
>	- Windows 8以降のworkstationではこのサービスはデフォルトで無効化
>	- Adminは管理目的でこのサービスを有効化している可能性もある
>	- Windows  Server 2012 R2、2016(1607)、2019(1809)、2022(21H2)ではデフォルト有効


## PsLoggedOnの使用方法

ADのdnshostnameで列挙したマシンに対して実行し、ドメイン内のマシンにログオン中のユーザーがいないかを確かめる
	[コンピューターオブジェクトの列挙 w/ PowerView](../Cheatsheet/🪟Windows/Active%20Directory/🔍AD%20Enumeration.md#コンピューターオブジェクトの列挙%20w/%20PowerView)
```powershell
# 例：.\PsLoggedon.exe \\files04.corp.com
.\PsLoggedon.exe \\<dnshostname>
```

- 出力例①：CORP\jeffというユーザーがログオン中
```powershell
Users logged on locally:
     <unknown time>             CORP\jeff
Unable to query resource logons
```

- 出力例②：ログオンユーザーなし
	偽陽性の可能性有り：Remote Registry サービスが有効でない場合も同じ出力
	エラーが出力されていないなら、結果は信じてもいいかもしれない
	（偽陽性の可能性は少しある）
```powershell
No one is logged on locally.
Unable to query resource logons
```

- 出力例③：resource share経由でログオン中
	現在コマンドを実行しているマシンから、指定したcommon nameのマシンに対し、SMB共有等でログインしている
```powershell
Users logged on via resource shares:
     10/5/2022 1:33:32 AM       CORP\stephanie
```


