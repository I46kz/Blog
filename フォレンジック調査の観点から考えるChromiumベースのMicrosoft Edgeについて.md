本稿は次の記事を自分なりに日本語訳してみたものになります：
https://forensicfocus.com/News/article/sid=3870/  
筆者(Oleg Skulkin:@oskulkin)から許可をいただいた上で挙げさせていただいています。  
※私自身記事の内容について知らないことも多く、コメントアウトで書いていたり、別で分からないことリストでまとめています。  

<br>

# フォレンジック調査の観点から考えるChromiumベースのMicrosoft Edge  

先ごろ、Microsoft社がようやくChromiumベースのEdgeブラウザーをリリースしました。それによりESEデータベース<!--調べる-->に関しての見落としが出てくるようです。もちろん、ChromiumやChromeと同様のフォレンジックアーティファクトのセットがあるかもしれませんが、これについても確認する必要があります。さらに新たなEdgeブラウザはWindowsだけでなく、macOS・Android・iOSでも使用できます。  

Windows上でのEdgeデータは次の場所から入手することが出来ます：  

***C:\Users\\\<username>\AppData\Local\Microsoft\Edge\User Data\Default***  

ブックマークもしくは"お気に入り"から始めていきましょう。それらはブックマークと同様のファイル名のJSONファイルとして保存されます。あなたは任意のテキストエディターでそれを開くことが出来ます。そのタイムスタンプはWebKit形式（1601年1月1日00:00 UTCからの64bitマイクロ秒値<!--https://www.wdic.org/w/TECH/%E6%97%A5%E4%BB%98%E5%9E%8B-->）で保存されています。 

<br>

キャッシュは**キャッシュサブフォルダ**に保存され、インデックスファイル(index)・データブロックファイル(data_#)・データファイル(f_######)からなっています。NirSoftの[ChromeCacheView](https://www.nirsoft.net/utils/chrome_cache_view.html)を使用するとこれらのファイルを簡単に解析できます。  

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

ご覧の通り<!--As you can see出過ぎ問題。原文の書いた人の癖を感じる。。。-->、任意のテキストエディタで開くことができる様々なファイルとBlob オブジェクト<!--https://hakuhin.jp/js/file.html ｜ http://5509.hatenablog.com/entry/2013/04/26/012658 この辺参照。Binary Large Objectの略で、32KB以上のデータをJavaScriptで扱う際に使用する。らしい。。。-->があります。運が良ければ、magic bytes<!--(ファイルシグネチャにおける、すでにリスト化(例：https://en.wikipedia.org/wiki/List_of_file_signatures(Wikipediaを出典に出すのもあれかもしれない・・・))されているファイルの正しい形式を識別できるもののこと。)-->のあるいくつかのblobを見つけることができ、そこからダウンロードしたコンテンツ自体を取得することが出来ます。  

他に便利なディレクトリは **/Library/Cookies/** / サブフォルダーです。ここでは[EdgeCookiesParser](https://github.com/HikaruHikarin/EdgeCookiesParser)で解析することができる、**Cookies.binarycookiesファイル**をみつけることが出来ます。  

最後に、Androidも重要です。Androidにおける、Microsoft Edgeのデータを保持する方法はWindowsおよびMacOSと同じです。**/data/data/com.microsoft.emmx/app_chrome/Defaultフォルダ**を見ると全ての必要なファイルとSQLiteデータベースを確認できます。キャッシュは **/data/data/com.microsoft.emmx/cache/Cacheフォルダ**に保存されており、これはChromeCacheViewで解析することができます。  

ここまで読んでいただいて分かる通り、一番重要な検索データ<!--Browsing dataをどう訳すかね。-->の抽出はかなり単純ないくつかのSQLクエリを用いることで可能となります。SQLiteデータベースを扱っているため、フリーリスト<!--https://ja.wikipedia.org/wiki/%E5%8B%95%E7%9A%84%E3%83%A1%E3%83%A2%E3%83%AA%E7%A2%BA%E4%BF%9D 動的なメモリの割り当てのこと。メモリの未使用領域をあらかじめ分類しておくことで、いざ必要になった際にその要求されたものを満たす領域に素早く割り当てることができる。って解釈でいいんだよね・・・？-->と未割り当て領域を忘れてはいけません。ここからさらに多くのアーティファクトが見つかるかもしれず、そこには調査の鍵となる要素の含まれている場合もあります。


