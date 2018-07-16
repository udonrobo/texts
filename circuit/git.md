# Gitとは
Gitとはソースコード等テキストファイルを管理するためのシステムです。
変更点を記録しておきその地点まで戻ったり、複数人で開発して書いたコードを一つに統合したりといったことが出来ます。

# インストール
## Windows
`http://git-scm.com/downloads`

# 用語とか
## repository リポジトリ
プロジェクトのソースコード等が入っているディレクトリの事。
## commit
変更の記録。これを作っていく。メッセージを付けられる。変更履歴も見れる。
## upstream
大本のリポジトリ。基本的にこれを直接触ることはない。
## origin
自分が作業するためにupstreamからforkしたリポジトリ。ネットワーク上にある。これにコミットを送信しておき、upstreamにPull Requestを送る。
## fork
upstreamから自分の作業用にネットワーク上のリポジトリを作る操作。
## pull request
originの変更をupstreamに取り込むように申請すること。
## clone
ネットワーク上のリポジトリをローカルに持ってくること。
## branch
変更を記録していく枝。
masterが大本。大体はmasterからdevを生やし、devから更にブランチを生やしてそこで作業する。

# 基本操作
## 初期化
```
git init
```
大抵はGitHubでリポジトリを作ってからそれをcloneすることになるのでこの操作をすることはあまりない(と思う)
## ファイルの変更、追加、削除をcommit対象に追加する。
```
git add ファイル名
```
これで出来る。
```
git add -p ファイル名
```
でファイル内の変更を個別にadd出来る。
## commit対象からファイルを取り除く(addの取り消し)
```
git reset HEAD ファイル名
```
## commit
```
git commit
```
このコマンドを打つと自動的にエディタが立ち上がりコミットメッセージの入力を求められる。
```
git commit -m "コミットメッセージ"
```
でコミットメッセージをそのまま入力できる。
## branchを作る
```
git branch ブランチ名
```
## branchの一覧
```
git branch
```
## branchの移動
```
git checkout ブランチ名
```
```
git checkout -b ブランチ名
```
でブランチを作ってそのブランチに移動できる。
## 変更がありcheckout出来ないみたいな事を言われた
```
git stash
```
一時的に変更を避難させる。
stashした変更の一覧は
```
git stash list
```
stashした変更の適用は
```
git stash apply 変更名(stash@{?})
```
stashした変更の削除は
```
git stash drop 変更名
```
stashした変更の適用と削除を同時に
```
git stash pop 変更名
```
## 変更をoriginに送信
```
git push origin ブランチ名
```

## ブランチAをブランチBに合流させる
```
git merge ブランチA ブランチB
```
# 開発の進め方。
## 基本
GitHub上でupstreamをforkする。
自分のoriginをcloneする。
ローカルでdevからbranchを切る。
バグ修正ならfix-???、機能追加ならfeature-???など
作ったブランチをoriginにpushし、GitHub上でPull Requestを送る。
## upstreamが変更されたら
```
git fetch upstream
git merge upstream/dev dev
git merge dev devから生やしてるブランチ
```

# 困ったとき
 * ググる
 * 誰か先輩を呼ぶ
