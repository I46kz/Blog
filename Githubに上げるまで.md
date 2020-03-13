Githubに上げるまで  

人に3時間くらいかけて心底丁寧に教えてもらったんだけれども、それでもメモをできてなかった部分ややり方が分からなかった部分もあったので、再度調べたりしつつやる。

<!--教えてくれた人@HASH1da1さん-->
参考にしたサイト  
- [GitHubの導入〜基本操作 for Windows](https://qiita.com/Kenta-Okuda/items/c3dcd60a80a82147e1bf)
- [【メモ】GitHubでgit clone〜git pushまで](https://qiita.com/nt_tn/items/c5ea999a2638e03ee418)  

教えてもらったときはMacだったのだが、WinでのGitの環境を整えるため、ひとまずGitを[インストール](https://git-scm.com/download/win)  

ローカルのディレクトリをリモートのリポジトリとしてあげる場合  
- https://qiita.com/pinevillage/items/4ccd3c3f59adcccdd7ef  

CRLFに関するメッセージについて  
warning: LF will be replaced by CRLF in "ファイル名".
The file will have its original line endings in your working directory
と出るのはなぜ？
- https://qiita.com/ritsuka/items/e4e1b9aa36b83886ae17
- https://tutorialmore.com/questions-1583513.htm  

WindowsでTrueにしたい場合
git config core.autocrlf true

そもそもGit bushをWindowsでは使ったほうが良さそう
- https://ikuten.com/github-newrepoditory   
- https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E5%9F%BA%E6%9C%AC-%E5%A4%89%E6%9B%B4%E5%86%85%E5%AE%B9%E3%81%AE%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%B8%E3%81%AE%E8%A8%98%E9%8C%B2  

pushする際にerror:failed to push some refsとなる  
- https://qiita.com/kazuki0714/items/ceda3a6721a9a99082de  
- https://qiita.com/yoshixj/items/6441ab2cd6bc367e607d  



