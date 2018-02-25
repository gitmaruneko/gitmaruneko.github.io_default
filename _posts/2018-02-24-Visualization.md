---
layout: post
title:  "於Jenkins可視化測試結果"
image: ''
date: 2018-02-25 11:50
tags:
- visualization
- Test
- Automation
- Jenkins
description: ''
categories:
- WORK
serie: 
---

 
規劃測試架構迄今進展還算順利,  
之前有提到由於團隊的測試工作幾乎手動操作  
而且在進行完一連串的手動測試後還必須人工整合測試結果,  
將文字檔整理成EXCEL寄出,   
每天進行這樣重複的工作, 我光聽就覺得時間都被浪費掉了  
所以**自動化**在現階段絕對是必要工作  

## 現況
由於先前已經有一套寫好的測試腳本  
所以在Jenkins上設定完任務, 便可以按時間自動運行  
但是, 如何呈現測試結果才是重點  
目前的測試結果是用文字檔的方式呈現, 如下圖A :  

<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/00_01.jpg"}}" width="656" height="263" alt="">
</figure>

再看看另一張圖B : 
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/00_02.jpg"}}" width="658" height="262" alt="" >
</figure>

有發現到問題嗎?
1. 圖A中測項全過>被判定為成功執行,  
   圖B中有**失敗**測項>仍被判定為成功.
2. 結果是否不太直覺? 因為測試工程師們必須從這些文字內確認測試結果.

第一個狀況主要因為測試腳本成功結束執行  
故Jenkins判定任務為成功  
所以在這點, 真正的做法係必須針對輸出LOG的內容去判斷  

而第二個狀況則必須針對測試結果做進一步的視覺化處理工作

## Jenkins套件法力無邊

針對前述問題一,  可以安裝下列套件做處理
1. [**Log Parser Plugin**](https://plugins.jenkins.io/log-parser)  
**Log Parser**可以在Jenkins運行結束後,  
根據使用者設定的關鍵字去分析LOG內容, 並且整合統計結果

#### 設定 : 
* 首先設定要偵測的關鍵字, 例如 : 測項內有 **FAIL**字眼, 便判定結果為error
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/00.jpg"}}" width="485" height="186" alt="" >
</figure>  
* 然後打開 Jenkins任務, 設定前述關鍵字檔案的存放位置,  
可以設定公用規則也可依個別專案設定規則, 以下兩個選項建議勾選 :  
　　* **Mark build Failed on Error** : 任何設定為Error層級的關鍵字被偵測到,  
　　　則此任務便判定為Fail  
　　* **Show log parser graphs** : 於任務頁面顯示log parser的統計結果,  
　　　(在此可得知這次測項結果走勢)

<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/01.jpg"}}" width="1016" height="346" alt="" >
</figure>
測項結果走勢示意圖 : 
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/02.jpg"}}" width="560" height="243" alt="" >
</figure>

設定後, 運行測試看看使用成果, 發現FAIL測項果然被高亮了,  
而且任務如預期被判定為**失敗**
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/03.jpg"}}" width="1144" height="484" alt="" >
</figure>

現在自動化測試的運行行為已經正常了, 但這樣仍然不夠,  
因為若有測項失敗, 工程師仍得花一些時間檢查LOG釐清原因,  示意圖
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/03_1.jpg"}}" width="592" height="104" alt="" >
</figure>
如果可以一眼就看到失敗原因, 想必會省下不少時間!  
這時可以搭配 [**Build Failure Analyzer Plugin**](https://plugins.jenkins.io/build-failure-analyzer)  
**Build Failure Analyzer** 這個套件可以根據使用者設定的字串分析LOG,  
當建置失敗, 便會分析LOG尋找關鍵字串, 找到後會在LOG高亮顯示結果,  
並且直接於建置頁面顯示

#### 設定
* 安裝套件後, 於Jenkins主頁面點選 **Failure Cause Management**  
* 在這個頁面可以看到所有設定為Failure Cause的選項, 點選 **Create New**
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/04.jpg"}}" width="718" height="311" alt="" >
</figure>
* 設定此Failure的名稱 > 輸入所要偵測的關鍵字串, (將字串以**.\***包起來)  
例如 : **.\*偵測字串.\***  
下圖為例, **Error: Unable to send alert signal**字串即為導致測試失敗的原因
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/03_1.jpg"}}" width="592" height="104" alt="" >
</figure>
所以輸入如下 : 
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/05.jpg"}}" width="350" height="805" alt="" >
</figure>
做完設定後, **若運行任務失敗**便會主動進行Failure Cause偵測,  
現在再次運行測試, 發現任務建置頁面出現了Identified problems告知失敗原因
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/06.jpg"}}" width="800" height="303" alt="" >
</figure>
點選**Indication 1**檢視失敗原因, LOG被高亮了!  
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/07.jpg"}}" width="732" height="158" alt="" >
</figure>

## 可視化工作

### 使用Pipeline讓測項可視化
上述的工作設定都還偏向文字呈現, 若要友善閱讀還是要有圖呈現較好  
以各測項為例, 可以使用**Pipeline**來設定測試工作  
成功失敗會直接以**<font color="red">紅</font><font color="green">綠</font>**顏色呈現  
在圖上也能看到每個測項所花費的時間, 還有平均時間, 整體來說方便很多  
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/08.jpg"}}" width="1188" height="302" alt="" >
</figure>
Pipeline script示意圖 :
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/08_02.jpg"}}" width="384" height="302" alt="" >
</figure>
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/08_01.jpg"}}" width="649" height="223" alt="" >
</figure>

### 測試結果報表可視化
解決完前述運行測試過程的相關工作, 來到最後一步  
要寄出給整個團隊的測試結果報表該如何處理?  
之前有提到, 測試工程師根據測試的結果文字檔整理出一份EXCEL表, 
再寄出給團隊, 這些作法有那些問題? 
其一 : 每次都要用對照文字檔和EXCEL表來整理, 時間太多? 何況人工實在太容易出錯  
其二 : 這份EXCEL表為local file, 只具備兩個生命週期  
一為測試工程師將EXCEL表存放於本地端, 一為Email內的附件  
沒有一個統一管理的地方, 當要調閱某次的測試結果都得撈很久  

所以就目前的狀況, 採用了 txt file 轉 XML file的處理方式  
XML file可以用HTML套用CCS樣式呈現於網頁, 也能轉為EXCEL表寄出  
當測試結束後, 便會自動運行 TXT > XML  > HTML的工作
然後將HTML嵌入EMail直接寄給團隊, 完全省略人工處理  

HTML所呈現的測試結果 : 
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/09.jpg"}}" width="921" height="193" alt="" >
</figure>

也可發布於建置頁面 (採用[**HTML Publisher Plugin**](https://plugins.jenkins.io/htmlpublisher))
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/10.jpg"}}" width="589" height="628" alt="" >
</figure>

#### 設定 
* 設定所要發布的HTML檔案路徑
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/12.jpg"}}" width="1010" height="534" alt="" >
</figure>

* 若要嵌入Email一同寄出, 需要將Conten Type調為HTML格式  
  並且於Content內指定File path變數名稱
<figure class="foto-legenda">
	<img src="{{ "/assets/img/maruIMG/20180824/11.jpg"}}" width="692" height="393" alt="" >
</figure>

採用上述方式, 是我目前能想到較為方便的做法  
由於團隊所使用的測試腳本是由**PERL**所撰寫  
(還是極度舊版的PERL, 所以沒什麼套件可用)  
跟用 **JUnit**撰寫單元測試時, 只要做些簡單設定便有圖表可看真的是兩回事 (泣)  

### Share
* [convert TXT to XML](https://github.com/gitmaruneko/ShareUtil/tree/master/convertTXTtoXML)
* [convert XML to HTML](https://github.com/gitmaruneko/ShareUtil/tree/master/convertXMLToHTML)























