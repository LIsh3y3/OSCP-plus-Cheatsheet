
- [XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## Table of contents

- トグルを開くとそれぞれのペイロードへのリンクが載っている
![[Pasted image 20231225180502.png | 200]]

---
## Event handlers

- Copy tags to clipboardはすべてのタグをクリップボードにコピーする。これをIntruder等で使用する
- その他のCopyも同様

- ブラウザの種類を選択することによって、そのブラウザで有効なevents、payloadsに絞ってコピーできる(tagは共通)
 ![[Pasted image 20231225181044.png | 100]]

- Tag, event, browserの種類を選択すると有効なPayloadがトップに表示される
![[Pasted image 20231225190613.png]]