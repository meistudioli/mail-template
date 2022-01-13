# mail-template

由於現在越來越多的 mail client 已經支援了 Dark mode，為了提供一致的瀏覽體驗，所以理論上 mail content 能夠支援 Dark Mode 的話，便能讓瀏覽體驗昇華至另一境地。但是由於 mail client 對 Dark mode 的支援度有所差異，便成了最大的阻礙。

目前 mail client 可以分成以下三種支援：

## CSS @media (prefers-color-scheme: dark)

與平常 front-end 撰寫 CSS 相同，可以針對 Dark mode 下不同的 style，當 user 切換 view mode 的時候便能立即反應，不過目前完整支援的 mail client 不多，僅有 Apple Mail 以及 Outlook series app。

```html
<style>
body {
  background-color: black;
  color: white;
}

@media screen and (prefers-color-scheme: light) {
  body {
    background-color: white;
    color: black;
  }
}
</style>
```

## Partial color invert
透過 Color invert 讓 mail client 針對 text / background 設定的顏色進行反轉，比方說 #fff <-> #000，好處就是 mail content 不需要做任何額外的設置，便能立即享有 Chrome auto dark mode like 的效果。不過由於它是 partial support，所以有些顏色的轉換可能不如預期，需要特別注意。目前 Android Gmail APP 便是屬於 Partial color invert。

```html
<head>
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
<style>
:root {
  color-scheme: light dark;
  supported-color-schemes: light dark;
}
</style>
</head>
```


## Full color invert
透過 Color invert 讓 mail client 針對 text / background 設定的顏色進行反轉，比方說 #fff <-> #000，好處就是 mail content 不需要做任何額外的設置，便能立即享有 Chrome auto dark mode like 的效果。目前 iOS Gmail APP 便是屬於 Full color invert。由於它是 full support，所以可以透過瀏覽器直接進行調校，可以免去反覆寄發 mail 測試的苦楚。

```html
<head>
<meta name="color-scheme" content="light dark">
<meta name="supported-color-schemes" content="light dark">
<style>
:root {
  color-scheme: light dark;
  supported-color-schemes: light dark;
}
</style>
</head>
```

## Roadmap
長遠來說，如果可以直接透過 @media (prefers-color-scheme: dark) 來進行設置的話，對 front-end engineers 來說掌握度最高，亦可任意渲染成 designer 所期待的樣式。不過要 full mail client 都支援的話，似乎還有很長一條路要，所以筆者建議先從 Color invert 的調適做起，掌握一些要點，讓當前的 mail content 於 dark mode 下瀏覽亦可讓 user 看得舒服為主要訴求。

以下為 Auction mail 於 iOS Gmail APP 的 light / dark mode 渲染

| light mode | dark mode | 
| --- | --- |
|![light_a](https://user-images.githubusercontent.com/10822546/148893295-33747f45-163a-49ba-a03e-d7f067499e21.PNG) | ![dark_a](https://user-images.githubusercontent.com/10822546/148893306-8a9fb8b2-38f1-4880-9bbf-bfbe3a3ba406.PNG)|
|![light_b](https://user-images.githubusercontent.com/10822546/148893364-e15c5af4-9237-4209-a960-6918ffce7f9a.PNG)|![dark_b](https://user-images.githubusercontent.com/10822546/148893380-89e08930-e3e6-4a61-afb8-63bc9fb2f90c.PNG) |
|![light_c](https://user-images.githubusercontent.com/10822546/148893513-3cd52f3e-b926-423a-a14b-7e6cf82f30a9.PNG) | ![dark_c](https://user-images.githubusercontent.com/10822546/148893524-f9fe6d75-2252-4167-a496-9e3433905899.PNG)|

透過 Color invert 渲染，大致上看起來還算舒服，剩下的就是讓它更好的調適。以下為一些設計要點整理：

1. Logo 或者 hero image 儘量使用具備 alpha channel 的圖片（透明背景 PNG），自然在不同 mode 的切換·便能無縫的融入其中，如果 logo 有貼近深色背景的設置則建議加上 white border，讓它呈現時可以更加清晰。
2. 顏色的使用上需要特別注意 Color Invert 後會不會造成很突兀的表現，比方說上圖可以看到黃色的膠囊按鈕，Color invert 後，便渲染出很詭異的按土黃色。所以在使用 color 時需要不斷的反覆進行調試，取得一個平衡值。
3. tag selector 要注意是否在 mail client 的 legal list，避免被 filter 造成樣式丟失（比方說 h1 於 Gmail 中會被 filter）。Diver 不可恥，能 work 最重要。
4. font-size 不傲期待繼承一事，儘可能在末梢 element 進行設置。也因為這樣，所以需要避免使用相對單位（em），因為我們不知道參照的對象內容是什麼。顏色方面的話除非是特意需要做加強的詞彙，不然儘量使用 #fff / #000，讓 APP 進行 color invert。
5. 避免使用 shorthand 語法，比方說 `background: url(https://blog.lalacube.com/mei/img/yau/logo.png) no-repeat 0% 0%/auto 100%;`，這是因為大部分的 mail client 都會有個 legal list 去 trim mail content，避免 mail style 污染回 page container，所以有可能因為它認不得就把屬性給 trim 掉了，因此儘量把它拆解成各個對應的屬性來設置。
6. 避免使用過於新穎的 CSS propertie，原因如上。
7. 由於測試上會反覆的寄發 email，最好有可以快速的調適 / 發送的 tool 來做，筆者建議使用 [Apps Script](https://script.google.com/home) 來進行發送。
```
function darkModeMail() {
  var htmlBody = HtmlService.createHtmlOutputFromFile('darkmode').getContent();

  MailApp.sendEmail({
    to: Session.getActiveUser().getEmail(),
    subject: 'Test Email markup - Dark mode -' + new Date(),
    htmlBody: htmlBody,
  });
}
```
8. 設定幾個主流目標：Apple Mail / Gmail / Yahoo Mail / Outlook 就好，不要囫圇吞棗什麼都要能支援，那只會搞死自己。

## Reference
- [Dark Mode for email marketing: how to make your emails look great](https://www.simpleviewinc.com/blog/stories/post/dark-mode-for-email-marketing-how-to-make-your-emails-look-great/)
- [The Basics of Email Dark Mode](https://www.mailgun.com/blog/email-dark-mode/)
- [Can I email](https://www.caniemail.com/search/?s=dark%20mode)
- [Dark mode with CSS](https://css-tricks.com/dark-modes-with-css/)
- [Using CSS in HTML Emails: The Real Story](https://css-tricks.com/using-css-in-html-emails-the-real-story/)
- [Apps Script](https://script.google.com/home)
