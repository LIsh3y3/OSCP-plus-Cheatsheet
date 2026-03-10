目的：正常を知ることで、異常かどうかの判断ができるようにする。

# 🪟Windows

## Common Services

| サービス名                                    |                                                                正式名称 / 略称 | 役割                                                  |
| ---------------------------------------- | -----------------------------------------------------------------------: | --------------------------------------------------- |
| AJRouter                                 |                                                   AllJoyn Router Service | ・ローカルネットワーク向けのデバイス検出と通信中継<br>・特定の IoT / UWP シナリオで利用 |
| ALG                                      |                                        Application Layer Gateway Service | ・NAT や一部プロトコルのパススルー支援<br>・古いアプリの互換性支援               |
| AppIDSvc                                 |                                                     Application Identity | アプリケーションアイデンティティの管理とアプリ制御                           |
| Appinfo                                  |                                                  Application Information | ・UAC 昇格のサポート<br>・管理者承認の処理                           |
| AppMgmt                                  |                                                   Application Management | ・レガシーなアプリ管理機能の提供<br>・一部 MSI ベースの配布で使用される            |
| AppReadiness                             |                                                            App Readiness | ・新規アプリの初回起動準備<br>・アップデート後のアプリ整合性処理                  |
| AppVClient                               |                                                   Microsoft App-V Client | 仮想アプリケーションの実行サポート                                   |
| AppXSvc                                  |                                                  AppX Deployment Service | ・ストアアプリのインストールと管理<br>・Windows Store の基盤             |
| AudioEndpointBuilder                     |                                                   Audio Endpoint Builder | ・オーディオデバイスの構成管理<br>・既定のオーディオエンドポイント提供               |
| Audiosrv                                 |                                                            Windows Audio | オーディオデバイスの再生/録音を提供                                  |
| AxInstSV                                 |                                                ActiveX Installer Service | ActiveX のインストール支援                                   |
| BFE                                      |                                                    Base Filtering Engine | ・ファイアウォールとパケットフィルタの基盤<br>・IPsec の一部機能を支援            |
| BITS                                     |                                  Background Intelligent Transfer Service | ・バックグラウンドファイル転送の提供<br>・Windows Update 等で使用される       |
| bthserv                                  |                                                Bluetooth Support Service | Bluetooth デバイス管理                                    |
| CDPSvc                                   |                                       Connected Devices Platform Service | デバイス検出と軽量データ共有                                      |
| CertPropSvc                              |                                                  Certificate Propagation | スマートカード証明書の読み込みと登録                                  |
| ClipSVC                                  |                                         Client License Service (ClipSVC) | ・Store ライセンス管理<br>・一部ストアアプリのライセンス検証                 |
| CoreMessagingRegistrar                   |                                                  CoreMessaging Registrar | UWP コアメッセージングの初期化支援                                 |
| CryptSvc                                 |                                                   Cryptographic Services | ・暗号サービスの提供<br>・証明書や署名の検証支援                          |
| CscService                               |                              Client Side Caching Service (Offline Files) | ・オフラインファイル同期の管理<br>・キャッシュの提供                        |
| defragsvc                                |                                                 Optimize Drives (Defrag) | ディスク最適化スケジュール実行                                     |
| DeviceAssociationService                 |                                               Device Association Service | ・デバイスペアリングの管理支援<br>・デバイス関連イベント処理                    |
| DeviceInstall                            |                                                   Device Install Service | ドライバーのインストールとセットアップ支援                               |
| DevQueryBroker                           |                                                      Device Query Broker | デバイス情報照会のブローカー                                      |
| Dhcp                                     |                                                              DHCP Client | ・IP アドレスの自動取得<br>・DHCP 更新処理                         |
| diagnosticshub.standardcollector.service |                                       Diagnostics Hub Standard Collector | アプリ/診断データの収集とロギング                                   |
| DispBrokerDesktopSvc                     |                                                   Display Broker Service | デスクトップ表示のブローカー処理                                    |
| DmEnrollmentSvc                          |                                     Device Management Enrollment Service | MDM 端末登録処理のサポート                                     |
| dmwappushservice                         |                                                  dmwappushsvc (WAP Push) | ・モバイルプッシュ/通知処理の受信サポート<br>・一部アプリ通知で使用                |
| Dnscache                                 |                                                               DNS Client | DNS 名前解決とキャッシュ管理                                    |
| DoSvc                                    |                                                    Delivery Optimization | ・Windows Update 等の分散ダウンロード支援<br>・ピアキャッシュ機能          |
| dot3svc                                  |                                                         Wired AutoConfig | 802.1X 有線認証の管理                                      |
| DPS                                      |                                                Diagnostic Policy Service | トラブルシューティングと診断ポリシーの提供                               |
| DsmSvc                                   |                                                     Device Setup Manager | デバイスセットアップとドライバー準備                                  |
| DsSvc                                    |                                                     Data Sharing Service | アプリ間のデータ共有機能                                        |
| EapHost                                  |                                       Extensible Authentication Protocol | 802.1X 認証や EAP モジュールの管理                             |
| EFS                                      |                                                   Encrypting File System | ファイル暗号化の提供                                          |
| EventLog                                 |                                                        Windows Event Log | ・イベントログの受信と保管<br>・イベントの読み出し基盤                       |
| EventSystem                              |                                                        COM+ Event System | COM イベントの配信と管理                                      |
| fdPHost                                  |                                         Function Discovery Provider Host | ネットワークデバイス検索のプロバイダホスト                               |
| FDResPub                                 |                                  Function Discovery Resource Publication | デバイスをネットワークに公開する機能                                  |
| FontCache                                |                                               Windows Font Cache Service | フォントキャッシュの管理と高速化                                    |
| FrameServer / FrameServerMonitor         |                                                             Frame Server | ・カメラ/ビデオフレームの中継と処理支援<br>・カメラ関連リソース管理                |
| GraphicsPerfSvc                          |                                             Graphics Performance Service | グラフィック性能の監視と最適化                                     |
| hidserv                                  |                                           Human Interface Device Service | HID デバイスの入出力処理支援                                    |
| HvHost                                   |                                                     Hyper-V Host Service | Hyper-V のホスト関連管理                                    |
| IKEEXT                                   |                                      IKE and AuthIP IPsec Keying Modules | IPsec / VPN の鍵交換とネゴシエーション                           |
| InstallService                           |                                                Windows Installer Service | ・MSI のインストール/アンインストール管理・ソフトウェア配布で使用                 |
| KPSSVC                                   |                                                        KpsSvcGroup サービス群 | 特定のパフォーマンス/管理グループで利用される支援サービス                       |
| KtmRm                                    |                            KtmRm for Distributed Transaction Coordinator | 分散トランザクションの管理                                       |
| LanmanServer                             |                                                         Server (SMB サーバ) | SMB ファイル/プリンタ共有の提供                                  |
| LanmanWorkstation                        |                                                 Workstation (SMB クライアント) | SMB クライアント機能の提供                                     |
| lfsvc                                    |                            Geolocation Service / Location Framework サポート | 位置情報の提供と管理                                          |
| LicenseManager                           |                                                          License Manager | ライセンス/アクティベーションの管理支援                                |
| lltdsvc                                  |                                     Link-Layer Topology Discovery Mapper | ネットワーク構成の検出支援                                       |
| lmhosts                                  |                                                    TCP/IP NetBIOS Helper | NetBIOS 名解決補助                                       |
| LSM                                      |                                                    Local Session Manager | セッションのライフサイクル管理                                     |
| MapsBroker                               |                                                      Maps Broker Service | マップデータと位置情報サービスの支援                                  |
| McpManagementService                     |                                                   MCP Management Service | モバイル接続/計画関連の管理                                      |
| mpssvc                                   |                                                         Windows Firewall | Windows ファイアウォールのコアサービス                             |
| MSDTC                                    |                            Microsoft Distributed Transaction Coordinator | 分散トランザクションの調整と復旧                                    |
| MSiSCSI                                  |                                        Microsoft iSCSI Initiator Service | iSCSI ストレージ接続の管理                                    |
| msiserver                                |                                                        Windows Installer | MSI パッケージのインストール管理                                  |
| NcaSvc                                   |                                           Network Connectivity Assistant | ネットワーク接続性の補助                                        |
| NcbService                               |                                                Network Connection Broker | ネットワーク接続のブローカー                                      |
| Netlogon                                 |                                                                Net Logon | ドメイン認証とログオン支援                                       |
| Netman                                   |                                                      Network Connections | ネットワークアダプタと接続の管理                                    |
| netprofm                                 |                                                     Network List Service | ネットワークプロファイルの管理                                     |
| NetTcpPortSharing                        |                                             Net.Tcp Port Sharing Service | WCF 等の TCP ポート共有支援                                  |
| NgcCtnrSvc / NgcSvc                      |                                             Windows Hello / NGC コンテナサービス | ・PIN や Windows Hello 関連の保管管理・認証要素の保護                |
| NlaSvc                                   |                                               Network Location Awareness | ネットワーク場所の検出とプロファイル切替                                |
| nsi                                      |                                          Network Store Interface Service | ネットワーク状態とストアの提供                                     |
| PcaSvc                                   |                                          Program Compatibility Assistant | 互換性の自動検出と対応支援                                       |
| PerfHost                                 |                                                         Performance Host | パフォーマンスカウンタとホスト支援                                   |
| pla                                      |                                                Performance Logs & Alerts | パフォーマンスログとアラート管理                                    |
| PlugPlay                                 |                                                            Plug and Play | デバイス検出とドライバロード                                      |
| PolicyAgent                              |                                                       IPsec Policy Agent | IPsec ポリシーの管理                                       |
| Power                                    |                                                                    Power | 電源設定とポリシーの管理                                        |
| PrintNotify                              |                                               Print Notification Service | 印刷通知とプリンタイベントの処理                                    |
| ProfSvc                                  |                                                     User Profile Service | ユーザープロファイルの読み込みと管理                                  |
| PushToInstall                            |                                                  Push To Install Service | リモートインストール支援                                        |
| QWAVE                                    |                                   Quality Windows Audio Video Experience | メディア品質の制御と QoS 支援                                   |
| RasAuto / RasMan                         | Remote Access Auto Connection Manager / Remote Access Connection Manager | ・VPN やダイヤルアップ接続の管理・接続自動化の支援                         |
| RemoteRegistry                           |                                                          Remote Registry | リモートからのレジストリ編集を許可する機能                               |
| RmSvc                                    |                                                         Resource Manager | リソースとレプリケーションの管理                                    |
| RpcEptMapper                             |                                                      RPC Endpoint Mapper | RPC エンドポイントのマッピング                                   |
| RpcLocator                               |                                            Remote Procedure Call Locator | RPC 名前解決支援                                          |
| RpcSs                                    |                                              Remote Procedure Call (RPC) | ・Windows の多くの基本機能の RPC 処理基盤<br>・RPC 呼び出しの仲介         |
| RSoPProv                                 |                                         Resultant Set of Policy Provider | グループポリシーの結果計算支援                                     |
| sacsvr                                   |                                        Security Accounts Controller サポート | セキュリティとアカウント管理の補助                                   |
| SamSs                                    |                                                Security Accounts Manager | ローカルアカウントとセキュリティデータの管理                              |
| SCardSvr                                 |                                                       Smart Card Service | スマートカード認証とミドルウェア支援                                  |
| ScDeviceEnum                             |                                             Smart Card Device Enumerator | スマートカードリーダの列挙と管理                                    |
| Schedule                                 |                                                           Task Scheduler | ・スケジュールタスクの実行と管理・自動実行ワークロードの基盤                      |
| SCPolicySvc                              |                                        Smart Card Removal Policy Service | スマートカード取り外しポリシーの管理                                  |
| seclogon                                 |                                                          Secondary Logon | 別ユーザーでのプロセス実行支援 (runas)                             |
| SecurityHealthService                    |                                          Windows Security Health Service | セキュリティコンポーネントのヘルス監視                                 |
| SENS                                     |                                        System Event Notification Service | システムイベント通知と状態管理                                     |
| SensorDataService / SensorService        |                                                          Sensor Services | センサー情報の取得と提供                                        |
| SessionEnv                               |                                      Remote Desktop Services Session Env | RDS セッション環境の管理                                      |
| SgrmBroker                               |                                           Smart Game Recording Broker など | ゲーム録画等のブローカー処理                                      |
| SharedAccess                             |                                        Internet Connection Sharing (ICS) | インターネット共有と NAT 提供                                   |
| ShellHWDetection                         |                                                 Shell Hardware Detection | リムーバブル機器の自動再生検出                                     |
| shpamsvc / smphost                       |                                             Smart Card / Security Host 系 | OS 機能支援                                             |
| SNMPTRAP                                 |                                                        SNMP Trap Service | SNMP トラップの受信                                        |
| Spooler                                  |                                                            Print Spooler | 印刷ジョブの管理                                            |
| sppsvc                                   |                                     Software Protection Platform Service | ライセンスとプロダクト保護                                       |
| SSDPSRV                                  |                                                    SSDP Discovery (UPnP) | ネットワーク上の UPnP デバイス検出                                |
| ssh-agent                                |                                             OpenSSH Authentication Agent | SSH キーのエージェント管理                                     |
| SstpSvc                                  |                                 Secure Socket Tunneling Protocol Service | SSTP VPN の管理                                        |
| StateRepository                          |                                                 State Repository Service | UWP アプリ状態の管理                                        |
| StiSvc                                   |                                          Still Image Acquisition Service | スキャナやカメラの取得処理                                       |
| StorSvc                                  |                                                          Storage Service | ストレージ管理機能                                           |
| svsvc                                    |                                          Storage Service / Small Service | ストレージ関連支援                                           |
| swprv                                    |                                            Software Shadow Copy Provider | VSS プロバイダのソフトウェア実装                                  |
| SysMain                                  |                                                     SysMain (Superfetch) | アプリ起動高速化とキャッシュ管理                                    |
| SystemEventsBroker                       |                                                     System Events Broker | システムイベントに基づく処理ブローカー                                 |
| TabletInputService                       |                                     Touch Keyboard and Handwriting Panel | タッチ入力と手書きパネルのサポート                                   |
| tapisrv                                  |                                                    Telephony API Service | テレフォニーAPI と TAPI サポート                               |
| TermService                              |                                            Remote Desktop Services (RDS) | RDP サーバー機能の提供                                       |
| Themes                                   |                                                                   Themes | 見た目とビジュアルスタイルの管理                                    |
| TieringEngineService                     |                                                   Tiering Engine Service | ストレージ階層化の管理                                         |
| TimeBrokerSvc                            |                                                      Time Broker Service | UWP アプリのバックグラウンド実行管理                                |
| TokenBroker                              |                                       Web Account Manager / Token Broker | Web アカウントトークンの管理                                    |
| TrkWks                                   |                                         Distributed Link Tracking Client | ファイルリンクの追跡                                          |
| TrustedInstaller                         |                             Windows Modules Installer (TrustedInstaller) | ・Windows コンポーネントのインストール/削除管理・Windows Update の一部を実行  |
| tzautoupdate                             |                                            Time Zone Auto Update Service | タイムゾーン自動更新                                          |
| UALSVC                                   |                                              User Access Logging Service | アクセスログの収集支援                                         |
| UevAgentService                          |                                                       UE-V Agent Service | ユーザー環境移行のエージェント                                     |
| UmRdpService                             |                         Remote Desktop Services UserMode Port Redirector | RDP のポートリダイレクト機能                                    |
| upnphost                                 |                                      Universal Plug and Play Device Host | UPnP デバイスのホスト                                       |
| UserManager                              |                                                             User Manager | ユーザーセッション管理                                         |
| UsoSvc                                   |                                              Update Orchestrator Service | 更新適用のオーケストレーション                                     |
| VaultSvc                                 |                                                 Credential Vault Service | 認証情報の保管と提供                                          |
| vds                                      |                                                     Virtual Disk Service | 仮想/物理ディスクの管理                                        |
| vm3dservice / vmic* / vmvss              |                                              VMware / Hyper-V ゲスト統合サービス群 | 仮想環境の統合機能提供                                         |
| VSS                                      |                                               Volume Shadow Copy Service | ボリュームシャドウコピーの管理                                     |
| W32Time                                  |                                                             Windows Time | 時刻同期（NTP）                                           |
| WaaSMedicSvc                             |                                             Windows Update Medic Service | Windows Update の修復と回復支援                             |
| WalletService                            |                                                           Wallet Service | デバイスのウォレット/支払い情報管理                                  |
| WarpJITSvc                               |                                                         Warp JIT Service | 実行最適化のための支援サービス                                     |
| WbioSrvc                                 |                                                Windows Biometric Service | 生体認証デバイスの支援                                         |
| Wcmsvc                                   |                                       Windows Connection Manager Service | 接続管理とメディア同期支援                                       |
| WdiServiceHost / WdiSystemHost           |                                                          Diagnostic Host | 診断情報の収集と実行環境提供                                      |
| Wecsvc                                   |                                                  Windows Event Collector | リモートイベント収集                                          |
| WEPHOSTSVC                               |                                                         WepHostSvc Group | エンタープライズ保護関連支援                                      |
| Wercplsupport / WerSvc                   |                                                  Windows Error Reporting | エラー報告とクラッシュダンプ収集                                    |
| WiaRpc                                   |                                            Windows Image Acquisition RPC | 画像取得とデバイス管理の RPC                                    |
| WinHttpAutoProxySvc                      |                                         WinHTTP Web Proxy Auto-Discovery | WPAD 自動プロキシ検出                                       |
| Winmgmt                                  |                                 Windows Management Instrumentation (WMI) | WMI 管理基盤                                            |
| WinRM                                    |                                                Windows Remote Management | PowerShell Remoting の基盤・リモート管理インターフェース              |
| wisvc                                    |                                      Windows Image Acquisition Service 等 | 画像/インストール支援                                         |
| wlidsvc                                  |                                     Windows Live ID Sign-in Assistant 相当 | Microsoft アカウント関連の支援                                |
| wmiApSrv                                 |                                                  WMI Application Service | WMI 拡張やアプリ用 WMI サポート                                |
| WPDBusEnum                               |                                       Portable Device Enumerator Service | ポータブルデバイスの列挙                                        |
| WpnService                               |                                       Windows Push Notifications Service | プッシュ通知の提供                                           |
| WSearch                                  |                                                 Windows Search / Indexer | ファイル検索とインデックスの構築                                    |
| wuauserv                                 |                                                           Windows Update | 更新プログラムの検出と適用                                       |

---

## Common process

| プロセス名 (Image Name)      | 正式名称 / 略称                                     | 役割                                                              |
| ----------------------- | --------------------------------------------- | --------------------------------------------------------------- |
| System Idle Process     | システム アイドル プロセス                                | ・CPUの空き時間を計測するプロセス<br>・通常、CPU使用率の残りの値を占める                       |
| System                  | System (Kernel)                               | ・Windows カーネルの主要なスレッドを実行<br>・ドライバのロードやメモリ管理の基盤                  |
| Registry                | Registry Service                              | ・レジストリ データの管理とキャッシュ<br>・システム全体の構成情報を保持                          |
| smss.exe                | Session Manager Subsystem                     | ・セッションの作成と管理（ログオンセッションの確立）<br>・システムの初期化プロセスの開始                  |
| csrss.exe               | Client Server Runtime Process                 | ・Win32 サブシステムのユーザーモード側を担当<br>・ウィンドウの作成やスレッド管理（通常、セッションごとに1つ存在）  |
| wininit.exe             | Windows Initialization Process                | ・サービスコントロールマネージャー（services.exe）などの起動<br>・システム全体の初期化（セッション0で実行）  |
| winlogon.exe            | Windows Logon Application                     | ・ユーザーのログオン・ログオフ・ロック画面を制御<br>・SAS（Ctrl+Alt+Del）の検知               |
| services.exe            | Service Control Manager (SCM)                 | ・システムサービスの開始、停止、および管理<br>・バックグラウンドで動くサービス群の親プロセス                |
| lsass.exe               | Local Security Authority Subsystem Service    | ・ユーザー認証（パスワード検証）の実行<br>・セキュリティ ポリシーの適用、トークンの生成                  |
| svchost.exe             | Service Host                                  | ・DLL（サービス）を実行するためのホストプロセス<br>・役割ごとに複数のインスタンスが同時に実行される           |
| fontdrvhost.exe         | User Mode Font Driver Host                    | ・ユーザーモードでのフォントレンダリング<br>・セキュリティのためにカーネルから分離されたフォント処理            |
| dwm.exe                 | Desktop Window Manager                        | ・デスクトップの描画と視覚効果（Aero、透過、アニメーション）<br>・GPUを利用したウィンドウ合成            |
| explorer.exe            | Windows Explorer                              | ・ユーザーインターフェース（タスクバー、スタートメニュー、デスクトップ）<br>・ファイルブラウザ機能の提供          |
| ShellExperienceHost.exe | Start / Shell Experience Host                 | ・スタートメニュー、カレンダー、時計などのユニバーサルアプリUI制御<br>・モダンなシェル機能のホスト            |
| SearchHost.exe          | Windows Search Host                           | ・Windows 検索機能（インデックス検索）のUIと処理<br>・タスクバーの検索ボックスに関連               |
| RuntimeBroker.exe       | Runtime Broker                                | ・Microsoft Store アプリのアクセス許可（カメラ、位置情報等）を管理<br>・サンドボックス内のアプリの権限監視 |
| conhost.exe             | Console Window Host                           | ・コマンドプロンプトやPowerShellのウィンドウ描画<br>・古いCUIアプリとシステムの仲介              |
| taskhostw.exe           | Host Process for Windows Tasks                | ・実行ファイル（.exe）形式ではないタスクの実行ホスト<br>・サインイン時に各種ライブラリを読み込む            |
| ctfmon.exe              | CTF Loader (IME)                              | ・日本語入力（IME）や手書き入力、音声認識の制御<br>・テキスト入力サービスをサポート                   |
| sihost.exe              | Shell Infrastructure Host                     | ・デスクトップの背景、通知、コンテキストメニューなどの基本UI要素のホスト<br>・UIの安定性を支える            |
| smartscreen.exe         | Windows Defender SmartScreen                  | ・不明なアプリや悪意のあるサイトからの保護<br>・ダウンロードファイルの安全性を検証                     |
| MsMpEng.exe             | Antimalware Service Executable                | ・Microsoft Defender のスキャンエンジン本体<br>・マルウェアの検知とリアルタイム保護           |
| NisSrv.exe              | Microsoft Network Realtime Inspection Service | ・ネットワーク経由の攻撃を検知する Defender の機能<br>・不正なネットワーク通信の監視               |
| spoolsv.exe             | Print Spooler Service                         | ・印刷ジョブのバックグラウンド処理<br>・プリンタードライバーとの通信                            |
| SgrmBroker.exe          | System Guard Runtime Monitor Broker           | ・OSの起動プロセスと整合性の監視<br>・ハードウェアベースのセキュリティ保護                        |
| Taskmgr.exe             | Task Manager                                  | ・タスクマネージャー本体<br>・プロセスの監視やシステムリソースの確認                            |

## Common Files / Directory

| ファイル/ディレクトリ名                                                              | 役割                                                                                                     |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| C:\Windows\debug\PASSWD.LOG                                               | ・Windowsのパスワード変更操作のログファイル<br>・パスワードの平文は記録されず、変更日時やユーザー名のみ記録<br>・システムが自動生成する標準的なログファイル                  |
| ~\AppData\Local\Microsoft\Edge\User Data\ZxcvbnData\3.0.0.0\passwords.txt | ・Microsoft Edgeのパスワード強度チェック用の辞書ファイル<br>・一般的な脆弱パスワードのリストを保存<br>・ユーザーの実際のパスワードは含まれない                     |
| ~\AppData\Local\Microsoft\Windows\WebCache                                | ・Internet ExplorerやEdgeのキャッシュデータベース<br>・閲覧履歴、Cookie、ダウンロード履歴などを保存<br>・WebCacheV01.datなどのESEデータベース形式    |
| C:\Windows\memory.dmp                                                     | ・ブルースクリーン（BSOD）発生時に生成されるメモリダンプファイル<br>・システム障害の診断用<br>・クラッシュ時のメモリ内容を保存するが通常は分析ツールが必要                    |
| ~\NTUSER.DAT                                                              | ・ユーザーごとのレジストリハイブファイル（HKEY_CURRENT_USER）<br>・ユーザー固有の設定や環境変数を保存<br>・システムが自動管理する標準ファイル                    |
| ~\AppData\Local\Temp                                                      | ・アプリケーションやシステムが一時ファイルを保存<br>・インストーラーの展開ファイルやキャッシュデータ<br>・定期的にクリーンアップされる標準的な一時領域                        |
| C:\hiberfil.sys                                                           | ・Windowsの休止状態（ハイバネーション）用ファイル<br>・メモリ内容を保存してシャットダウン時の状態を復元<br>・物理メモリとほぼ同サイズの隠しファイル                      |
| C:\pagefile.sys                                                           | ・Windowsの仮想メモリ（ページファイル）<br>・物理メモリが不足した際にディスクをメモリとして使用<br>・システムが自動管理する標準的なスワップファイル                      |
| C:$Recycle.Bin                                                            | ・削除されたファイルを一時的に保管するごみ箱<br>・各ユーザーのSIDごとにサブディレクトリを作成<br>・ファイルの復元が可能な標準的なシステムフォルダ                         |
| ~\AppData\Local\Microsoft\Windows\History                                 | ・Internet Explorerの閲覧履歴を保存<br>・訪問したWebサイトのURLと日時を記録                                                    |
| C:\Windows\Prefetch                                                       | ・アプリケーション起動を高速化するためのプリフェッチファイルを保存<br>・実行されたプログラムの履歴が分かるが、機密情報は含まれない<br>・フォレンジック調査では有用だが攻撃には利用しづらい      |
| ~\AppData\Local\Microsoft\Windows\INetCache                               | ・Internet ExplorerやEdgeのキャッシュファイルを保存<br>・閲覧したWebページの画像やスクリプトなど                                         |
| C:\Windows\System32\LogFiles                                              | ・Windowsの各種システムログを保存するディレクトリ<br>・HTTPエラーログ、ファイアウォールログなどを格納<br>・診断情報が主で直接的な認証情報は含まれない                   |
| doc                                                                       | ・アプリケーションのdocフォルダにはデフォルトで用意された操作方法などがあるだけ                                                              |
| vendor                                                                    | ・パッケージ管理ツールが、外部ライブラリ（SDK、フレームワークの本体、ユーティリティなど）をインストールする場所<br>・他人が作ったプログラムのコピー                          |
| SysWOW64                                                                  | ・Windows 32-bit On Windows 64-bit の略<br>・64ビット Windows 上で、32ビット用のプログラムを動かすための仕組み                       |
| WinSxS                                                                    | ・Windows Side-by-Side<br>・Windows のすべてのコンポーネントのマスターコピー<br>・基本的にはシステムファイル（.dll や .exe）の集まり              |
| C:\Program Files \Common Files                                            | ・標準的な共有ライブラリが多い<br>・書き込み管理者権限が必要                                                                       |
| C:\Windows\System32\Drivers\DriverData                                    | ・主にハードウェア設定データ<br>・**wi-fi**使用時は要注目                                                                    |
| C:\ProgramData                                                            | ・Microsoft：MS製品の共有データ<br>・Package Cache：インストーラーキャッシュ<br>・Application Data：旧Vista互換<br>・Adobe：Adobe製品設定 |

---
---

# 🐧Linux

## Common Files / Directory


| ファイル/ディレクトリ名                    | 役割                                                                                                                                                     | ペネトレーションテストの着眼点                                                                                                                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| .bash_logout                    | ・bashシェルからログアウト時に自動実行されるスクリプト  <br>・画面クリアやクリーンアップ処理などを記述  <br>・デフォルトでは最小限の設定のみ                                                                         | -                                                                                                                                                                     |
| .cache                          | ・アプリケーションの一時キャッシュデータを保存  <br>・削除しても再生成されるデータ  <br>・パフォーマンス向上のため各種プログラムが利用                                                                              | -                                                                                                                                                                     |
| snap                            | ・Snapパッケージの個人データを保存  <br>・インストールしたSnapアプリの設定やデータを格納  <br>・各Snapアプリごとにサンドボックス化されたデータ領域                                                                  | ・各Snapアプリの設定ファイル内の認証情報                                                                                                                                                |
| crash                           | ・Ubuntuなどのシステムでプログラムがクラッシュした際のクラッシュレポートを保存  <br>・`apport`サービスが自動的に生成  <br>・ファイル名は通常 `_path_to_program.UID.crash` の形式  <br>・デバッグ情報、スタックトレース、メモリダンプなどを含む | ・クラッシュレポート内にメモリダンプから機密情報（パスワード、トークン、秘密鍵）が漏洩していないか  <br>・エラーメッセージから内部のファイルパス、システム構成、バージョン情報を取得  <br>・開発環境の情報（デバッグシンボル、ソースコードパス）の露出  <br>・コアダンプファイルの有無確認（機密データが含まれる可能性） |
| /usr/share/mysql/dictionary.txt | ・MySQLにはvalidate_password というプラグインがある<br>・ユーザーがパスワードを設定する際に「簡単すぎるパスワード」を禁止する機能<br>・簡単なパスワードを定義している                                                     | 無視                                                                                                                                                                    |
| trustdb.gpg                     | 各公開鍵に対するユーザーの信頼レベルを記録したファイルで、重要ではない                                                                                                                    | -                                                                                                                                                                     |

---

## Cron

| 名称                   | 役割                                          |
| -------------------- | ------------------------------------------- |
| `apt-compat`         | パッケージリストの更新や不要なパッケージのクリーンアップ。               |
| `logrotate`          | 肥大化したログファイルを圧縮・分割・削除してディスク容量を節約する。          |
| `man-db`             | `man`コマンドで使うマニュアルのデータベースを更新・最適化する。          |
| `dpkg`               | インストール済みパッケージのバックアップ（`/var/backups`）を作成する。  |
| `apache2`            | Apacheの古いログや一時ファイルを整理する（Webサーバ入りなら一般的）。     |
| `update-notifier`    | 利用可能なセキュリティ更新があるかを確認し、通知を準備する。              |
| `sysstat`            | CPUやメモリの使用率などの統計情報を収集・記録する。                 |
| `php`                | PHPの古いセッションファイルを削除し、メモリやディスクを整理する。          |
| `fstrim`             | SSDのパフォーマンスを維持するために、未使用ブロックを破棄する。           |
| `e2scrub_all`        | ext4ファイルシステムの論理的なエラーをオンライン（マウント中）でチェックするツール |
| `popularity-contest` | インストールされているパッケージの利用統計を、匿名で開発元に送信するツール       |
| `mdadm`              | LinuxでRAIDを構築するツール                          |

---

## パッケージ

| パッケージ名                         | 役割                              |
| ------------------------------ | ------------------------------- |
| accountsservice                | ユーザーアカウント情報の問い合わせや管理            |
| adduser                        | ユーザーやグループをシステムに追加するツール          |
| apparmor                       | プログラムの権限を制限するセキュリティフレームワーク      |
| apport / apport-symptoms       | プログラムがクラッシュした際のレポート作成機能         |
| apt / apt-utils                | パッケージのインストールや更新を行う管理システム        |
| at                             | 指定した時間に一度だけジョブを実行するツール          |
| base-files / base-passwd       | OSのディレクトリ構造や標準的なパスワード定義         |
| cron                           | 指定した時間に定期的にジョブを実行するツール          |
| dbus / dbus-user-session       | プロセス間でメッセージをやり取りする通信システム        |
| debconf / debconf-i18n         | パッケージ設定時の対話形式インターフェース           |
| dpkg                           | パッケージ管理の最も低レベルな基礎ツール            |
| bash / bash-completion         | 標準的なコマンドシェルと、その補完機能             |
| coreutils                      | ls, cp, mv など、Linuxで必須の基本コマンド集  |
| dash                           | 高速に動作する軽量シェル（主に起動時に使用）          |
| debianutils                    | whichやtempfileなど、Debian系の基本ツール集 |
| diffutils                      | ファイル同士の差分を表示するツール               |
| bind9-dnsutils / host / libs   | digやhostなど、DNS情報の確認に使うツール       |
| curl                           | URL構文を用いてデータを転送するツール            |
| amd64-microcode                | CPUのバグを修正するためのマイクロコード           |
| busybox-initramfs / static     | 起動時（initramfs）などで使われる軽量ツール集     |
| cloud-init / cloud-guest-utils | クラウド環境でのインスタンス初期設定ツール           |
| console-setup / linux          | コンソールのフォントやキーボードレイアウト設定         |
| cryptsetup / bin / run         | ディスク暗号化（LUKS）を管理するツール           |
| dmidecode                      | BIOSからハードウェアのシリアル番号等の情報を取得      |
| dmsetup / dmeventd             | デバイスマッパー（論理ボリューム管理等）の制御         |
| bc                             | コマンドラインで使える計算機                  |
| bcache-tools                   | SSDをHDDのキャッシュにするための管理ツール        |
| bsdmainutils / bsdutils        | カレンダーや列表示など、伝統的なユーティリティ集        |
| btrfs-progs                    | btrfsファイルシステムの管理ツール             |
| bzip2 / cpio / dosfstools      | 圧縮解凍や、FAT形式ファイルシステムの操作ツール       |
| ca-certificates                | Web通信等の信頼性を確認するための証明書集          |
| bolt                           | Thunderboltデバイスの管理ツール           |
| command-not-found              | 入力したコマンドがない時にパッケージを提案する機能       |

## マウント仮想ファイルシステム

| 分類        | ファイルシステム / マウント名                                  | 無視してよい理由                |
| --------- | ------------------------------------------------- | ----------------------- |
| OS・プロセス情報 | `sysfs`, `proc`, `debugfs`, `tracefs`, `configfs` | カーネル内部やプロセス情報を表示するための窓口 |
| デバイス・端末   | `udev`, `devpts`                                  | デバイスファイルや仮想端末の管理用       |
| メモリ・一時領域  | `tmpfs`, `hugetlbfs`, `mqueue`                    | RAM上の作業領域。再起動で中身が消える    |
| コンテナ・管理   | `cgroup`, `snapd`, `nsfs`                         | リソース制限やパッケージ隔離用の管理領域    |
| 特殊・その他    | `binfmt_misc`, `fusectl`                          | バイナリ互換性やユーザー空間FSの制御用    |

## デーモン

| デーモン名                       | 役割（何をしているか）                   |
| --------------------------- | ----------------------------- |
| accounts-daemon.service     | ユーザー情報の管理（アイコンや言語設定など）。       |
| apparmor.service            | アプリケーションの権限を制限するセキュリティ機能のロード。 |
| apport.service              | システムエラー時の自動デバッグレポート作成。        |
| atd.service                 | atコマンドによる「単発の予約タスク」の実行。       |
| blk-availability.service    | ストレージデバイスの可用性管理。              |
| console-setup.service       | コンソールのフォントや文字コードを起動時に設定。      |
| cron.service                | crontabによる「定期的なタスク」の実行。       |
| dbus.service                | プロセス間で情報をやり取りするための通信バス。       |
| finalrd.service             | シャットダウン時に使う最小限のルートディレクトリの作成。  |
| fwupd-refresh.service       | ファームウェア更新情報の自動チェック（失敗しやすい）。   |
| getty\@tty1.service         | 端末（TTY）でのログイン画面の提供。           |
| keyboard-setup.service      | キーボードレイアウト（JIS/USなど）を起動時に設定。  |
| kmod-static-nodes.service   | 起動時に必要なデバイスノード（/dev/）の作成。     |
| lvm2-monitor.service        | 論理ボリューム（LVM）の監視。              |
| ModemManager.service        | モデムデバイスの管理。                   |
| multipathd.service          | ストレージへの複数経路（マルチパス）の管理。        |
| networkd-dispatcher.service | ネットワーク接続状態の変化に応じたスクリプトの実行。    |
| open-vm-tools.service       | VMwareなどの仮想環境との連携ツール。         |
| polkit.service              | システム全体の権限（認可）管理。              |
| rsyslog.service             | システムログの収集と記録。                 |
| setvtrgb.service            | コンソールのカラー構成（配色）の設定。           |
| snapd.apparmor.service      | Snapアプリ用のAppArmor設定の読み込み。     |