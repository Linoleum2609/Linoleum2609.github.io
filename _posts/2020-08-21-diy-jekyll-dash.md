---
layout: post
title:  "訂製你的 Jekyll Dash"
date:   2020-08-21
tags: Jekyll Theme DIY
---

# [Jekyll Bash Without Plugins](https://github.com/MyCodingSucks/jekyll-dash-without-plugins) 訂製說明  

[Jekyll Dash](https://github.com/bitbrain/jekyll-dash) 是 [bitbrain](https://github.com/bitbrain) 編寫的一款 Jekyll 主題，該主題默認帶有 [jekyll-tagging](https://github.com/pattex/jekyll-tagging) 和 [liquid-md5](https://github.com/pathawks/liquid-md5) 兩個第三方插件，而 [GitHub 官方文件](https://docs.github.com/en/github/working-with-github-pages/about-github-pages-and-jekyll#plugins) 顯示，這些第三方插件是不可以被使用的。bitbrain 給出的方法是[先使用 CI 構建](https://github.com/bitbrain/jekyll-dash#using-this-theme-directly-on-github-pages)（說實話我不知道 CI 是啥，我猜想那個方法是在其他地方構建後再上傳），但看上去~~略顯~~複雜。因此，我參考一堆文檔（但對小白有用的就那幾個），把 Jekyll Dash 改成了[沒有第三方 plugins 的版本](https://github.com/MyCodingSucks/jekyll-dash-without-plugins)，而這篇文章就是對這個更改的說明與解釋。  

Jekyll 中一大核心是 [Liquid](https://shopify.github.io/liquid/)。Liquid 是一種模板語言（Template Language）。作為圈外人，我們只需知道在 Jekyll 中，正是因為 Liquid，我們才有可能如此簡單地修改部落格的排版，而不影響整體的外觀。這說明如果你使用其他主題，這裡大部分也是適用的，只需要注意位置就行了。  

Liquid 的詳細語法可以參考其官方文檔。一般來說，在一堆 HTML 中看到一堆花括號和百分號的十有八九是 Liquid。  

既然都能翻到這了，用 GitHub Pages 搭建部落格的過程我就不廢話了。下面就是一些更改的說明。

## 自訂相關簡介  
當你 fork 或在本地構建完成打開網頁欣賞完後，我猜第一件事就是想要修改相關介紹。在 `_config.yml` 中很容易找到 Title 與 Description 的部分。那麼那些該死的圖標與頭像呢？  

Well。首先你要放好你所需的圖片，比如我的就放在 `assets` 裡面。  

要修改 Description 旁邊的頭像，你需要修改 `_includes` 檔案夾裡面的 `author.html`，在 `<img>` 標籤中，把 `src=""` 中的指向換成自己的就行了。   

要修改網站圖標（就是在瀏覽器標籤顯示的那個，當然如果你的瀏覽器標籤不顯示圖像就另說了），同樣地，在 `_includes` 中，把 `head.html`（該文件事實上就是網站的 `<head>` 部分）中 `href="{{ "/assets/favicon.png" | relative_url }}"` 改為指向你的圖標的位置。

## 底部菜單  

然後你看到了底部的菜單。你可能會想「sh*t，我不需要再 About Me 了」，或者你有其他的 ideas。總之你覺得這不符合你。  

首先，你可以自建一個你喜歡的檔案夾，或者也可以用我建的 `bottom_bar` ，在裡面加入你需要的文件。  

![files](/pics/2020-08-21/files.png)

然後想一下，你想放在哪裡。如果你想把他改到上面的話，複製下面的代碼到 `head.html` 中，記得讓他對應你喜歡的位置：  

```html
<p>|--------------------------------------------------|</p>
		
<style>
	ul {
		list-style-type: none;
		overflow: hidden;
			
    margin-left: -17px;
		margin-top: -25px;
		margin-bottom: -25px;
			
		padding-left: 7px;
    
		font-size:30px;
  }	
	
	li {
		float: left;
	}
	
	li a {
		display: block;
		text-align: center;
		padding: 14px 16px;
		text-decoration: none;
  }
</style>
  
<ul>
	<li><a class="active" href="/bottom_bar/tags.html">tags</a></li>
	<li><a href="/bottom_bar/about.html">about</a></li>
	<li><a href="/bottom_bar/friends.html">friends</a></li>
</ul>

<p>|--------------------------------------------------|</p>
```

然後再 `foot.html` 中刪除掉與之相同的代碼。再把文字與鏈接換成你的。  

如果你喜歡在下面，那就留他啦。

## Tag 頁面  
我個人不建議修改與 tag 相關的東西，因為關聯太多了。如果你一定要修改，請繼續看下去。  

底部的 Tag 頁，其實算一個 tag cloud，你可以嘗試塞一些 tag 看看。  

如果你足夠細心，會發現我 tag 部分的代碼都是抄[這裡](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/)的，十分感謝原作者的貢獻。  

所有代碼塊都有註釋，形式如 `{% comment %} balabala {% endcomment %}`。我只對訂製部分做些解釋好了。  

打開 `tags.html` （如果你在訂製底部菜單時把它刪了，你可以回到原 repo 查看），你可以很清楚地看到代碼分四塊。而且註釋中也把作用說得明明白白的。  

假設你喜歡 [bitbrain 那樣的樣式](https://bitbrain.github.io)（即 tag 直接放在頁面底部），我們以此為例說明。  

把 `tags.html` 前三塊的代碼複製，加到 `_layouts/home.html` 的**最後面**（請不要塞到中間某個地方，塞在最前面或最後面）。這樣 tag 應該就會出現在所有文章的最後面。  這三塊代碼是檢測並處理 tags，然後把它們都堆在一起。  

最後一塊代碼是把所有 tag 都列出，然後標出所有使用該 tag 的文章。把它放去你喜歡的地方，在 `_layouts/post.html` 中作相關改動就行了。

## Disqus 評論功能  
**友情提醒：因為某些眾所週知的原因，Disqus 無法在大陸使用。**   

首先你需要去 https://blog.disqus.com 註冊帳號，套餐拉到最下選免費的就好了。  

跟著它設定，**記得填好 Shortname**，它相當於一個 ID 而且**不能更改**。  

![disqus](/pics/2020-08-21/disqus.png)

回到 `_config.yml`，在 disqus 的 shortname 欄中填入你的 Shortname。**然後在 `url` 中填入部落格的網址**。   

![disqusconf2](/pics/2020-08-21/disqusconf2.png)

![disqusconf1](/pics/2020-08-21/disqusconf1.png) 

然後你也可以去開啟 Reactions，比如我朋友的長這樣。。。  

![reactions](/pics/2020-08-21/reactions.png)

## Google Analytics  
**如果無法使用 Google 的服務，不妨嘗試下 [Bing Webmaster](https://www.bing.com/toolbox/webmaster/)**。  

在 Google Analytics 設定完成後，你可能會得到一段 JS 代碼。當然你也可以在 `管理 > 追蹤資訊 > 追蹤程式碼` 中找到。  

![ganaly](/pics/2020-08-21/ganaly.png)

複製，把它粘貼到 `_includes/head.html` 中。等待一段時間就可以看到 Google Analytics 的追蹤了。

## 一點點小建議  
請不要忘記使用萬能的 Google，已經有許多大佬寫過教程了，本垃圾也是看了一堆教程才折騰出來的。  

如果在修改過程中遇到困難，不訪按下 F12 或者看一下[現成的參考](https://github.com/MyCodingSucks/MyCodingSucks.github.io)。GitHub 中也有無數模板可供參考。  

如果你有任何疑問，可是試著聯繫本垃圾，我會提供力所能及的幫助。而且因爲本人語文很差，時常表達不清，如有歧義，請指正。  

請大膽嘗試，不過要記得存個備份wwwww
