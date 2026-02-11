# Nmap

```zsh
PORT   STATE SERVICE REASON  VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
```

---

# Enumeration

## rpcdump

```zsh
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Retrieving endpoint list from 192.168.115.111
Protocol: [MS-RSP]: Remote Shutdown Protocol 
Provider: wininit.exe 
UUID    : D95AFE70-A6D5-4259-822E-2C84DA1DDB0D v1.0 
Bindings: 
          ncacn_ip_tcp:192.168.115.111[49664]
          ncalrpc:[WindowsShutdown]
          ncacn_np:\\OSCP[\PIPE\InitShutdown]
          ncalrpc:[WMsgKRpc0955A0]

Protocol: N/A 
Provider: winlogon.exe 
UUID    : 76F226C3-EC14-4325-8A99-6A46348418AF v1.0 
Bindings: 
          ncalrpc:[WindowsShutdown]
          ncacn_np:\\OSCP[\PIPE\InitShutdown]
          ncalrpc:[WMsgKRpc0955A0]
          ncalrpc:[WMsgKRpc095F91]

Protocol: N/A 
Provider: N/A 
UUID    : D09BDEB5-6171-4A34-BFE2-06FA82652568 v1.0 
Bindings: 
          ncalrpc:[csebpub]
          ncalrpc:[LRPC-0b03d68480bd270d61]
          ncalrpc:[LRPC-085a0d22fa2e471fc8]
          ncalrpc:[LRPC-1c8436fdc1b7514149]
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]
          ncalrpc:[LRPC-085a0d22fa2e471fc8]
          ncalrpc:[LRPC-1c8436fdc1b7514149]
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]
          ncalrpc:[LRPC-1c8436fdc1b7514149]
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]
          ncalrpc:[LRPC-067dee2eff45b1f3e1]
          ncalrpc:[LRPC-6b4f0facb47844b1a6]

Protocol: N/A 
Provider: N/A 
UUID    : 697DCDA9-3BA9-4EB2-9247-E11F1901B0D2 v1.0 
Bindings: 
          ncalrpc:[LRPC-0b03d68480bd270d61]
          ncalrpc:[LRPC-085a0d22fa2e471fc8]
          ncalrpc:[LRPC-1c8436fdc1b7514149]
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

...

Protocol: N/A 
Provider: N/A 
UUID    : 88ABCBC3-34EA-76AE-8215-767520655A23 v0.0 
Bindings: 
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 76C217BC-C8B4-4201-A745-373AD9032B1A v1.0 
Bindings: 
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 55E6B932-1979-45D6-90C5-7F6270724112 v1.0 
Bindings: 
          ncalrpc:[LRPC-80904044aba899cdc3]
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 857FB1BE-084F-4FB5-B59C-4B2C4BE5F0CF v1.0 
Bindings: 
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : B8CADBAF-E84B-46B9-84F2-6F71C03F9E55 v1.0 
Bindings: 
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 20C40295-8DBA-48E6-AEBF-3E78EF3BB144 v1.0 
Bindings: 
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 2513BCBE-6CD4-4348-855E-7EFB3C336DD3 v1.0 
Bindings: 
          ncalrpc:[LRPC-e5e3f5dad50a6bb7ec]
          ncalrpc:[OLE41F02C9B3211B2722FBE65F7DEFD]
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 0D3E2735-CEA0-4ECC-A9E2-41A2D81AED4E v1.0 
Bindings: 
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : C605F9FB-F0A3-4E2A-A073-73560F8D9E3E v1.0 
Bindings: 
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 1B37CA91-76B1-4F5E-A3C7-2ABFC61F2BB0 v1.0 
Bindings: 
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 8BFC3BE1-6DEF-4E2D-AF74-7C47CD0ADE4A v1.0 
Bindings: 
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 2D98A740-581D-41B9-AA0D-A88B9D5CE938 v1.0 
Bindings: 
          ncalrpc:[LRPC-8953aacdd42937fcd2]
          ncalrpc:[actkernel]
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 0361AE94-0316-4C6C-8AD8-C594375800E2 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 5824833B-3C1A-4AD2-BDFD-C31D19E23ED2 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : BDAA0970-413B-4A3E-9E5D-F6DC9D7E0760 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 3B338D89-6CFA-44B8-847E-531531BC9992 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 8782D3B9-EBBD-4644-A3D8-E8725381919B v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 085B0334-E454-4D91-9B8C-4134F9E793F3 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: N/A 
UUID    : 4BEC6BB8-B5C2-4B6F-B2C1-5DA5CF92D0D9 v1.0 
Bindings: 
          ncalrpc:[umpo]

Protocol: N/A 
Provider: sysntfy.dll 
UUID    : C9AC6DB5-82B7-4E55-AE8A-E464ED7B4277 v1.0 Impl friendly name
Bindings: 
          ncalrpc:[LRPC-943e959e22550d4b79]
          ncalrpc:[IUserProfile2]
          ncalrpc:[LRPC-57f3c772f3dbd7b141]
          ncalrpc:[LRPC-c86a084f2e7fcd4f19]
          ncalrpc:[senssvc]
          ncalrpc:[LRPC-cc642a38ada79a927a]

Protocol: N/A 
Provider: N/A 
UUID    : A500D4C6-0DD1-4543-BC0C-D5F93486EAF8 v1.0 
Bindings: 
          ncalrpc:[LRPC-b82541089020656368]
          ncalrpc:[LRPC-067dee2eff45b1f3e1]

Protocol: N/A 
Provider: N/A 
UUID    : E40F7B57-7A25-4CD3-A135-7F7D3DF9D16B v1.0 Network Connection Broker server endpoint
Bindings: 
          ncalrpc:[LRPC-fc053138c508ca64fa]
          ncalrpc:[OLE59F0DDF4F24E040240FA77BAED7C]
          ncalrpc:[LRPC-13e7ed3ea20e621d40]
          ncalrpc:[LRPC-6b4f0facb47844b1a6]

Protocol: N/A 
Provider: N/A 
UUID    : 880FD55E-43B9-11E0-B1A8-CF4EDFD72085 v1.0 KAPI Service endpoint
Bindings: 
          ncalrpc:[LRPC-fc053138c508ca64fa]
          ncalrpc:[OLE59F0DDF4F24E040240FA77BAED7C]
          ncalrpc:[LRPC-13e7ed3ea20e621d40]
          ncalrpc:[LRPC-6b4f0facb47844b1a6]

Protocol: N/A 
Provider: N/A 
UUID    : 5222821F-D5E2-4885-84F1-5F6185A0EC41 v1.0 Network Connection Broker server endpoint for NCB Reset module
Bindings: 
          ncalrpc:[LRPC-13e7ed3ea20e621d40]
          ncalrpc:[LRPC-6b4f0facb47844b1a6]

Protocol: [MS-EVEN6]: EventLog Remoting Protocol 
Provider: wevtsvc.dll 
UUID    : F6BEAFF7-1E19-4FBB-9F8F-B89E2018337C v1.0 Event log TCPIP
Bindings: 
          ncacn_ip_tcp:192.168.115.111[49665]
          ncacn_np:\\OSCP[\pipe\eventlog]
          ncalrpc:[eventlog]

Protocol: N/A 
Provider: N/A 
UUID    : DF4DF73A-C52D-4E3A-8003-8437FDF8302A v0.0 WM_WindowManagerRPC\Server
Bindings: 
          ncalrpc:[LRPC-a494467d96c17d35ce]
...

Protocol: [MS-SAMR]: Security Account Manager (SAM) Remote Protocol 
Provider: samsrv.dll 
UUID    : 12345778-1234-ABCD-EF00-0123456789AC v1.0 
Bindings: 
          ncacn_ip_tcp:192.168.115.111[49679]
          ncalrpc:[samss lpc]
          ncalrpc:[SidKey Local End Point]
          ncalrpc:[protected_storage]
          ncalrpc:[lsasspirpc]
          ncalrpc:[lsapolicylookup]
          ncalrpc:[LSA_EAS_ENDPOINT]
          ncalrpc:[LSA_IDPEXT_ENDPOINT]
          ncalrpc:[lsacap]
          ncalrpc:[LSARPC_ENDPOINT]
          ncalrpc:[securityevent]
          ncalrpc:[audit]
          ncacn_np:\\OSCP[\pipe\lsass]
```

## rpcclient

```zsh
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ rpcclient $TargetIP -U "" -N
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED

```

- sashaで接続してみる
```zsh
rpcclient 192.168.115.111 -W oscp.exam -U spayne -P 'Voltaic1992'
```
↑上記は誤った認証情報


```zsh
# -wはoscpでも試行
┌──(koshi㉿kali)-[~/PEN-200/Beta]
└─$ rpcclient 192.168.115.111 -W oscp -U sasha -P 'Voltaic1992' 
Failed to open /var/lib/samba/private/secrets.tdb
Failed to set machine account: NT_STATUS_INTERNAL_ERROR
```

---

# Screenshot


