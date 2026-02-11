- nurses.csvにはusernameっぽいリストがあるも、ADのユーザーではなさそう
```csv
;;;;;;
;Addison;Mueller;;;;
;Miley;May;;;;
;Aaliyah;Ortiz;;;;
;Marcus;Vaughan;;;;
;Harmony;Davenport;;;;
;Denzel;Garrison;;;;
;Lamar;Vance;;;;
;Lauren;Bright;;;;
;Lia;Griffith;;;;
;Nathalia;Mosley;;;;
;Carly;Dudley;;;;
;Aspen;Drake;;;;
;Walter;Kirk;;;;
;Reid;Whitney;;;;
;Carl;Mcintyre;;;;
;Konner;Atkinson;;;;
;Seth;Herman;;;;
;Marisa;Cabrera;;;;
;Cindy;Gibbs;;;;
;Demarion;Bernard;;;;
;Isaias;Prince;;;;
```

---

# SRV22

- JENKINSとIISが動作しているため、LFIの可能性はある
- martinの認証情報が、web.configのAPIに記載されていたことも気になる
	- →以下のディレクトリにローカルサービスでいくとか？
```xml
...
    <add key="ApiURL" value="https://...../servlets/AssetServlet" />
    <add key="ApiUserName" value="b.martin" />
    <add key="ApiPassword" value="MartiniAllNight222" /> 
...
```


- management.propertiesのポート番号が空白のとき、どれがデフォルトポート？



- snmpにアップロードして、responder?