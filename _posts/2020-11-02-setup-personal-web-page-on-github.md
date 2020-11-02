---
layout: post
title: 建立自己的github.io網頁
---

## fork範本與發布網頁

github有提供一個[官方範本](https://github.com/github/personal-website)，若要使用範本，直接fork範本repo即可，再把專案名稱改成`username.github.io`後，瀏覽器網址打上`username.github.io`就可看到精美的網頁。至此基本上已經完成網頁部署了！(現在寫網頁好快啊)

什麼，好醜? 那就來更換版型吧！網路上有很多基於jekyll技術的網頁範本，任君挑選，像是[這個](https://jamstackthemes.dev/ssg/jekyll/)。使用方法也是fork範本repo到你的github就對了。 (copy&paste就是這樣advance)

fork好範本後，接著把範本repo複製到你的電腦上

```
git clone https://github.com/{username}/{repo_name}.git
```

等修改工作都在本機端完成後，再push到github上，就完成了一個網頁修改的cycle了。

記得好多年前無名小站關站時，還要特地把所有寫過的文章備存下來，但多年下來也不知道遺失在哪顆硬碟裡了。

現在這種靜態網頁修改發布的流程就可以避免這種情況，因為寫文章、修改版面都在自己的本機端完成，再投過git發布到git server上面，哪天筆電不見了，再從遠端git server抓下來就好。

## 設定jekyll環境

為了在本機上編輯網頁，必須安裝Ruby的jekyll和bundler套件。安裝Ruby請至[官網下載](https://rubyinstaller.org/downloads/)。

jekyll是一個用Ruby寫成的套件，用來從pain text生成靜態網頁文本，而為了產生靜態網頁的過程中，jekyll會使用許多不同的Ruby套件輔助，這些套件稱為Gem，而bundler則是為了維護套件相依性所寫稱的另一個Gem套件。

```
gem install bundler jekyll
```

安裝好後，進入範本repo，透過bundler安裝所需要的gem套件。

```
cd {repo_name}
bundle install
```

在本機端開啟server後，瀏覽器網址打上`localhost:4000`即可看到精美網頁。

```
bundle exec jekyll serve
```

## 以Lanyon範本為例

以上是針對有使用gem套件的範本repo的大致設定流程。但為了設定本網頁所使用的Lanyon範本可是費了不少功夫...
拜google大神之賜，找到了這篇[過來人苦主的參考文章](https://www.loumarven.dev/2020/02/23/getting-a-good-old-jekyll-theme-to-work-on-gitlab-pages/)

當你把Lanyon fork到你的github，也把範本repo更名為username.github.io後，你會發現網頁排版走掉了，問題在於當把範本repo push到github上後，github端會透過自己的jekyll server產製靜態網頁到`public`資料夾下，正好覆蓋了Lanyon放在`public`資料夾下的css檔案，把public資料夾更名為assets即可正常瀏覽。

原始Laynon範本並沒有使用gem套件，所以只需要

```
jekyll build
jekyll serve
```

即可。

## 編輯更新網頁

編輯jekyll網頁所需基本了解可參考jekyll上的[step by step](https://jekyllrb.com/docs/step-by-step/01-setup/)

* 在本機端編輯網頁時，jekyll會在目錄`_site`下依據設定檔與資料產出靜態網頁。
* 主要設定在設定檔`_config.yml`。
* `_layouts`中放有產製網頁所使用的框架。
* 目錄`_includes`放有會被其他網頁常常拿去用的內容。
* 目錄`_posts`中放置新增markdown檔`XXX.md`，編輯的靜態網頁要放在這裡。
* 目錄`_data`可放置像是social-media設定...等資料。

每次存檔後jekyll會自動更新靜態網頁，若是修改_config.yml，則必須重啟jekyll服務才會看到更新


enjoy! :)

## 參考資料

* [Get started building your personal website](https://github.com/github/personal-website)
* [[Ting’s筆記Day2] 在Github用Jekyll創建自己的blog](https://ithelp.ithome.com.tw/articles/10198964)
* [How to build a blog using Github, Jekyll and Lanyon](https://www.nikhita.dev/build-blog-using-github-jekyll)
* [Getting a Good Old Jekyll Theme to Work on Gitlab Pages](https://www.loumarven.dev/2020/02/23/getting-a-good-old-jekyll-theme-to-work-on-gitlab-pages/)
* [Including Social Media Icons on Jekyll through the use of Data Files](https://jreel.github.io/social-media-icons-on-jekyll/)