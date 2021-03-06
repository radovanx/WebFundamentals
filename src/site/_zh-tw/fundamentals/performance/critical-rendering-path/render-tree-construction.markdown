---
layout: article
title: "轉譯樹狀結構的建構、版面配置和繪製"
description: "CSSOM 樹狀結構和 DOM 樹狀結構會結合成轉譯樹狀結構，用於計算每個可視元素的版面配置，並作為繪製過程的輸入參數，在螢幕上轉譯各個像素。 如要達成最佳轉譯成效，關鍵就在於確實執行最佳化流程的每個步驟。"
introduction: "CSSOM 樹狀結構和 DOM 樹狀結構會結合成轉譯樹狀結構，用於計算每個可視元素的版面配置，並作為繪製過程的輸入參數，在螢幕上轉譯各個像素。 如要達成最佳轉譯成效，關鍵就在於確實執行最佳化流程的每個步驟。"
article:
  written_on: 2014-04-01
  updated_on: 2014-09-18
  order: 2
collection: critical-rendering-path
authors:
  - ilyagrigorik
key-takeaways:
  render-tree-construction:
    - DOM 樹狀結構和 CSSOM 樹狀結構合併成轉譯樹狀結構。
    - 轉譯樹狀結構只包含轉譯網頁所需的節點。
    - 版面配置會計算每個物件的精確位置和大小。
    - 繪製是最後一步，將會以最終的轉譯樹狀結構做為輸入參數，在螢幕上呈現各個像素。
notes:
  hidden:
    - "順帶一提，請注意「visibility: hidden」和「display: none」不同。 前者會隱藏元素，但這個元素仍會佔據版面配置中的相應空間 (其實就是一個空白方塊)；而後者 (display: none) 會直接從轉譯樹狀結構中徹底移除元素，該元素不僅會消失，而且也不再屬於版面配置的一部分。"
---
{% wrap content%}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
</style>

在上一節中，我們介紹了物件模型的建構方式，也就是根據輸入的 HTML 和 CSS 建構 DOM 樹狀結構和 CSSOM 樹狀結構。 不過，這是兩個互相獨立的物件，分別涵蓋文件的不同層面：一個描述內容，另一個則是描述套用於文件的樣式規則。 那麼瀏覽器是如何將兩者結合起來，並在螢幕上呈現各個像素呢？

{% include modules/takeaway.liquid list=page.key-takeaways.render-tree-construction %}

對瀏覽器而言，第一步就是將 DOM 樹狀結構和 CSSOM 樹狀結構合併成「轉譯樹狀結構」，讓這個結構不僅包含網頁上所有可見的 DOM 內容，同時也包含各個節點的 CSSOM 樣式資訊。

<img src="images/render-tree-construction.png" alt="DOM 樹狀結構和 CSSOM 樹狀結構會合併成轉譯樹狀結構" class="center">

為了建構轉譯樹狀結構，瀏覽器大致需要執行下列步驟：

1. 從 DOM 樹狀結構的根節點開始，瀏覽每個可見的節點。
  * 某些節點完全無法察覺 (例如 script 標記、meta 標記等)。由於轉譯的輸出內容中不會反映這些節點，因此會被省略。
  * 某些節點透過 CSS 隱藏起來，在轉譯樹狀結構中也會被省略。以上述示例的 span 節點說明，由於該節點透過顯式規則設定了「display:none」屬性，因此不會出現在轉譯樹狀結構中。
1. 為每個可見的節點找到相符的 CSSOM 規則，並套用這些規則。
2. 發送可見的節點，包括內容和計算的的樣式。

{% include modules/remember.liquid list=page.notes.hidden %}

最終輸出的轉譯樹狀結構不僅包含螢幕上顯示的所有可見內容，同時也包含相應的樣式資訊。我們就快要大功告成了！  **有了轉譯樹狀結構，我們就能進入「版面配置」階段。**

到目前為止，我們已經計算了哪些節點應為可見節點及其計算的樣式，但是尚未計算節點在裝置的 [檢視區]{{site.fundamentals}}/layouts/rwd-fundamentals/set-the-viewport.html中的準確位置和大小。這就是在「版面配置」階段 (有時也稱為「重新編排」) 所要做的工作。

為了計算出每個物件的準確大小和位置，瀏覽器從轉譯樹狀結構的根節點開始瀏覽，以計算網頁上每個物件的幾何形狀。 下面就讓我們看一個簡單的例子：

{% include_code _code/nested.html full %}

以上網頁的內文包含兩個巢狀嵌套的 div：第一個 div (上層元素) 將節點的顯示大小設定為檢視區寬度的 50%，第二個 div (包含在上層元素中) 將寬度設定為上層元素的 50%，即檢視區寬度的 25%！

<img src="images/layout-viewport.png" alt="計算版面配置資訊" class="center">

版面配置過程的輸出結果稱為「盒子模型」，這個模型可精確說明每個元素在檢視區中的準確位置和大小：所有相對度量單位都會轉換為螢幕上的絕對像素位置。

知道哪些節點可見、計算的樣式和幾何形狀之後，我們終於可以將這些資訊傳遞到最後一個階段，將轉譯樹狀結構中的每個節點轉換為螢幕上的實際像素。這個步驟通常稱為「繪製」或者「點陣化」。

到目前為止，您都理解了嗎？ 上述每個步驟都需要瀏覽器進行大量的工作，這也表示可能經常需要消耗相當長的時間。 幸好，Chrome DevTools 可以協助我們深入瞭解上述各個階段。 讓我們檢視原始「hello world」示例中的版面配置階段：

<img src="images/layout-timeline.png" alt="在 DevTools 中評估版面配置" class="center">

* 在時間軸中透過「Layout」事件捕捉轉譯樹狀結構的建構、定位和大小的計算。
* 版面配置完成後，瀏覽器會觸發「Paint Setup」和「Paint」事件，將轉譯樹狀結構轉化為螢幕上的實際像素。

轉譯樹狀結構的建構、版面配置和繪製所需的時間取決於文件的大小、套用的樣式，當然還有執行文件的裝置：文件越大，瀏覽器需要完成的工作就越多；樣式越複雜，繪製所需的時間就越長 (例如，繪製單色「成本較低」，而計算和呈現陰影的「成本較高」)。

完成上述所有步驟後，我們的網頁終於呈現在檢視區了！

<img src="images/device-dom-small.png" alt="完工顯示的 Hello World 網頁" class="center">

讓我們快速回顧一下瀏覽器執行的所有步驟：

1. 處理 HTML 標記，產生 DOM 樹狀結構。
2. 處理 CSS 標記，產生 CSSOM 樹狀結構。
3. 將 DOM 樹狀結構和 CSSOM 樹狀結構合併為轉譯樹狀結構。
4. 對轉譯樹狀結構進行版面配置，計算每個節點的幾何形狀。
5. 在螢幕上繪製各個節點。

我們的示範網頁看起來也許很簡單，但是非常費工！ 如果修改了 DOM 或 CSSOM，您覺得將會發生什麼情況？ 如此一來，我們就必須重複上述步驟，以確定需要在螢幕上重新呈現的像素。

**所謂最佳化關鍵轉譯路徑，就是儘可能縮短上述第 1 步到第 5 步所花費的總時間。** 時間縮短後，我們就可以儘早在螢幕上呈現內容，還可以縮短初次轉譯後螢幕刷新的時間間隔，也就是提高互動式內容的更新速率。

{% include modules/nextarticle.liquid %}

{% endwrap%}

