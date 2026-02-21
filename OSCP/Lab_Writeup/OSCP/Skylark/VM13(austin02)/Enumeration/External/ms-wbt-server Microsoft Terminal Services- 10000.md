>Microsoft Terminal Services（ターミナル サービス）は、Windows Serverの画面やアプリケーションをネットワーク経由で離れたPCから操作できる技術です。Windows Server 2008 R2以降は「[リモート デスクトップ サービス](https://www.google.com/search?q=%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88+%E3%83%87%E3%82%B9%E3%82%AF%E3%83%88%E3%83%83%E3%83%97+%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9&rlz=1C1HKFL_jaJP1199JP1199&oq=Microsoft+Terminal+Services+%E3%81%A8%E3%81%AF&gs_lcrp=EgZjaHJvbWUyBggAEEUYOdIBCDEyNzNqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8&mstk=AUtExfDy8Le0V_tlNsY7q7qDXXNKreKibCcoWVi1z8FIlFI6nkn1e8VFzvjZElAf_uCpaEKIje_QKeGU5_nI7C9ZLVMk8fKMbLQpvFV2YlRY6a1TRBdy7MvushvOvaJCPZxB5L4gxUKbmJpeyyvDTyMALvLmPBT5C_y0v4vrPdNdcpahuW5anuMihzmtQhxxCPvMvu9LdSZoDCeWAoYJ1XC3qTu65IhF6JAJV0MSIWCFctt0hpkonR38Ca6kyiK89SJGCfFZCBZKaVi0c0qlVA1uXSwK&csui=3&ved=2ahUKEwjNorzApNqSAxVQafUHHWO0DbQQgK4QegQIARAB) (RDS)」に名称が変更されました。

→ただのRDPっぽい。
# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
10000/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-02-14T12:18:10+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=austin02.SKYLARK.com
| Not valid before: 2025-10-10T01:42:09
|_Not valid after:  2026-04-11T01:42:09
```

---


# Enumeration

- usermanualでRDWebというブラウザのパスと、RDWebのパスワードを入手したので、それでRemoteApp接続
- 


---

# Screenshot


