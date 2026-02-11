- 関連ノート：
	- [[135,593 - MSRPC]]

# 目的・シチュエーション

目的：
- ターゲットのRPCサービスのポート情報を収集し、次の攻撃対象（例：NFS, rusersd）を見つける。
- NULLセッションでの接続確認とユーザー列挙。

シチュエーション：
- LinuxもしくはWindowsマシン

---

# RPCサービスの列挙

バナー取得（手動）
```zsh
nc -nvC $TargetIP 111
```

サービスの簡易一覧（概要）
```zsh
rpcinfo -s $TargetIP
```

RPCサービスごとのポートを列挙
```zsh
rpcinfo -p $TargetIP
```

---

# NULLセッションでのユーザー列挙

```zsh
rpcclient -U "" $TargetIP
```

ログイン後のコマンド例（対話モード）
```zsh
rpcclient $> enumdomusers
rpcclient $> queryuser 0x[RID]
```

その他のコマンドは[[135,593 - MSRPC]]を参照