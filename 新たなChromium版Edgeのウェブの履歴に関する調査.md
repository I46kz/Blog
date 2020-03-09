https://www.foxtonforensics.com/blog/post/investigating-web-history-in-the-new-edge-chromium-browser
このページの翻訳。

新たなChromium版Edgeのウェブの履歴に関する調査  

2020年1月15日にMicrosoft社はChromiumをベースとしたEdgeブラウザーの安定版を初めてリリースしました。これは、Windows 7/8/8.1/10とmacOSに対して互換性があります。  

EdgeはChromeを筆頭としたオープンソースであるChromiumブラウザに基づいたWebブラウザに関する開発リストに参加しています。そしてこのことはEdgeが閲覧履歴をChromeとほぼ同じようなフォーマットで保存するようになったことを意味します。Windows10では、通常Edgeのプロファイルは次のディレクトリに保存されています：  
*C:\Users(ユーザー)\\\<username>\AppData\Local\Microsoft\Edge\User Data\Default*

最新の安定版のEdgeとChromeで作成された全てのSQLiteデータベースをSQLite Examinerを使用して比較したところ、顕著に違いがあったのはWeb Data SQLiteデータベース内だけでした。

Web Dataデータベースには、HTMLの入力フォームなどに使用する様々な自動入力データが格納されます。前回の記事<!--リンク貼って-->で、フォーム履歴をWebページのURLにリンクする方法を説明しました。そのおかげにより、これらのアーティファクトからこれまでよりも多くの情報(値)を得ることが出来ました。  
現在のChromeは自動入力データをVARCHER列にプレーンテキストで保存していますが、他方Edgeは自動入力データをBLOB列に保存していることがわかります。  

このBLOB列への保存という変更は、Webデータベースに保存する前にEdgeが自動入力データ(オートフィルデータ)を暗号化するためであることが確認できました。暗号化の方法は、ChromeがパスワードおよびCookieの暗号化に使用している方法と同様です。Windowsにおいては、Data Protection API(DPAPI)を使用して行われます。現状、Chromeを始めとした他のChromiumベースのブラウザが自動入力データの暗号化についてEdgeに追従するかは不明です。しかしもし追従する場合、アクセスするのが更に難しい重要なアーティファクトとなります。  
<!--DPAPIとは：http://bit.ly/3cxXlZ4-->

新たなEdgeブラウザの固有の機能の一つにInternet Explorer(IE)モードがあります。これにより、Internet Explorerを使用してレンダリングするWebページをEdgeで開くことが出来ます。  

IEモードによって作成されたアーティファクトについていくつかのテストを行い確認したところ、IEモードでレンダリングされたページはEdgeとInternet Explorerの両方のWeb履歴上に記録されていることを発見しました。  

したがって、たとえEdgeブラウザのみを使用していたとしても、調査時にはInternet Explorerの履歴が見つかってしまう場合もある。  

Browser History Examiner(BHE)<!--URL入れといて-->とフリーツールが更新されChromiumベースのEdgeブラウザを新たにサポートするようになりました。旧バージョンのEdgeについてもサポートは続いています。そのため、Edgeの旧バージョンと新しいバージョンのどちらの履歴も含むマシンを解析している場合にはこれらのツールは新旧両バージョンの履歴をキャプチャして読み込みます。  

Browser History Examinerの無料トライアルについては、ダウンロードページを御覧ください。  

<br><br>
ここからは：
https://forensicfocus.com/News/article/sid=3870/  

フォレンジックの観点から考えるChromiumベースのMicrosoft Edge  

先ごろ、Microsoft社がようやくChromiumベースのEdgeブラウザーをリリースしました。それによりESEデータベース<!--調べる-->に関しての見落としが出てくるようです。もちろん、ChromiumやChromeと同様のフォレンジックアーティファクトのセットがあるかもしれませんが、これについても確認する必要があります。さらに新たなEdgeブラウザはWindowsだけでなく、macOS・Android・iOSでも使用できます。  

Windows上でのEdgeデータは次の場所から入手することが出来ます：  

***C:\Users\\\<username>\AppData\Local\Microsoft\Edge\User Data\Default***  

ブックマークもしくは"お気に入り"から始めていきましょう。それらはブックマークと同様のファイル名のJSONファイルとして保存されます。あなたは任意のテキストエディターでそれを開くことが出来ます。そのタイムスタンプはWebKit形式（1601年1月1日00:00 UTCからの64bitマイクロ秒値<!--https://www.wdic.org/w/TECH/%E6%97%A5%E4%BB%98%E5%9E%8B-->）で保存されています。 

<br>

キャッシュは**キャッシュサブフォルダ**に保存され、インデックスファイル(index)・データブロックファイル(data_#)・データファイル(f_######)からなっています。NirSoftのChromeCacheView<!--リンク頼む-->を使用するとこれらのファイルを簡単に解析できます。  

Cookieは、**Cookie**と呼ばれるSQLiteデータベースに保存されます。Cookieが必要です。該当のクエリを次に示します：<!--cookieクエリ画像-->  

ご覧のように、**datetime関数**<!--http://bit.ly/2xfGJVF-->を使用してWebKit形式のタイムスタンプを簡単に変換することができます。  

Microsoft Edgeでダウンロードされたファイルの情報は**History SQLiteデータベース**から取得できます。  

もうひとつ便利なテーブルは**urls**です。繰り返しになりますが、シンプルなクエリを使用して訪問したサイトとタイムスタンプに関する情報を取得することができます。  

Edgeはプロファイル・ロケーション・カード番号といった自動入力(オートフィル)情報を**Web Dataデータベース**内に保存します。保存された資格情報は、**Login Dataデータベース**に保存されます。**loginsテーブル**内のURLとそれに関連したログインデータを見つけることができます。  

しかしながら、全てのパスワードは暗号化されています。NiftsoftのChromePassを使って復号化を試みることができます。このツールを使用すると、実行中のシステムまたは外部ドライブからパスワードを回復できます。エビデンス(証拠物)をどれぐらい簡単にマウントすることができるのかはここでは記しませんが、例えばFTK Imagerを使用して外部のドライブとして認識させます。必要なのはWindowsのユーザープロファイル(User Profile)<!--Windows Profileというのが原文。これでいいとは思うけど、違うかも。-->のパスワードのみです。  

その結果、オリジナルとアクセス用のURL<!--Origin URLとAction URLの違い-->、ユーザー名、パスワードなどの情報をプレーンテキストで取得し、さらに作成日時も知ることができます。  

プログレッシブウェブアプリケーション（PWA）はChromium Edgeの主要な機能の一つです。それを使用することで、デバイス上のウェブサイトについてウェブアプリケーションとして"インストール"することができます。より厳密に言うならば<!--In factの翻訳-->プロファイルディレクトリとアプリケーションIDを因数として取得し、アプリケーションシェル（static template<!--static templateをそのまま静的テンプレートって訳すのもなんか違う気がした。：https://dixq.net/forum/viewtopic.php?t=16674とか https://qiita.com/hmito/items/9d928322ca978319da59とか-->）を実行して、マニフェストファイルに記述されているURLから必要な動的コンテンツをロードするmsedge_proxy.exeがあります。

そのマニフェストファイルは **Extensions\\<App_ID>** のサブフォルダに保存されます。<!--自分の会社PCにはExtensionsのフォルダ自体が作成されていなかった。Edgeで拡張機能を入れてないからか・・・？-->  

同じフォルダーには、新しく追加された拡張機能のソースコードが含まれています。各拡張機能には、一意のIDで名前が付けられた独自のサブフォルダーがあります。  

MacOS上のEdgeファイルは非常に似ており、以下にあります：  

**/Users/\<username>/Library/Application Support/Microsoft Edge/Default**

ご覧の通り、ブックマーク、アクセスしたURL、ダウンロード、Cookieなどに関する情報は対応するファイルとSQLiteデータベースに保存されるため、前述の手法を使用してこのデータを取得できます。  

MacOS上では、キャッシュは **/Users/\<username>/Library/Caches/MicrosoftEdge/Default/Cache**フォルダーに個別に保存されることご注意ください。しかし、ChromeCacheViewを使用して解析できます。  

次の目的物はiOSです。全てのEdgeファイルは以下に保存されます：

**/private/var/mobile/Containers/Data/Application/\<UUID>**  

したがって、Microsoft EdgeとUUIDが一致する必要があります。どうやるのか？簡単です！必要になのは **/private/var/mobile/Library/FrontBoard/**以下にある、**applicationState.db**だけです。  

**application_identifier_tabテーブル**から正しいIDを見つけることから始めてみましょう。この場合、com.microsoft.msedgeのIDは121です。これにより見つかったIDを使用して**kvsテーブル**を確認し、 **application_identifier列**をフィルタリングすることができます。value列にはバイナリのplistが含まれており、エクスポートする必要があります。これは、例えばDBite for SQLiteを使用してこのタスクを解決できます。エクスポートすると、自分の気に入っているplistビューアで調べることができます。  

これにより、このMicrosoft EdgeのUUIDが565EC255-F158-48E1-83C5-D426BC60D22Dであることが確認でき、アプリケーションのデータを簡単に見つけることができます。  

はじめに、アクセスの履歴が保存されているDocumentsサブフォルダーに配置された**Office Cache**のSQLiteデータベースを確認できます。アクセスしたURLは**ZONLINESEARCHHISTORYテーブル**にApple NSDate形式のタイムスタンプで保存され、次のクエリで取得できます。  

**OfflineCacheデータベース**には、ブラウザの追加のブックマークとデータも保存されるため、SQLiteの同じDBブラウザを使用してそれらを確認することもできます。

アクセスの履歴に加えて、**Library/Caches/WebKit/NetworkCache/Version 14/Records/<Website_ID>/Resourceサブフォルダー**をチェックして、ダウンロードされたコンテンツについての見解を少しだけ得ることができます。









