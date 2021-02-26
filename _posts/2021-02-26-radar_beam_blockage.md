---
layout: post
title: 雷達能看到多低的雲?! 使用wradlib套件繪製雷達地形遮蔽圖
---

{% include image.html url="/assets/img/1920px-Cloud_types_en.svg.png" description="不同高度發展之各種雲狀。資料來源:wikipedia" %}

會下雨的雲主要是積雨雲(Cb)與雨層雲(Ns)，前者導因於大氣中垂直對流旺盛，把下層水氣往上層帶凝結所致，發展範圍可從500公尺至對流層頂(中緯度處~10公里)，後者可以是對流能量不夠強，以致衝不高所致，又或是伴隨著強烈對流系統出現之層狀降雨，發展範圍可從500公尺至5,500公尺。

參考資料: [收集水氣的濃積雲](http://scimonth.blogspot.com/2014/09/blog-post_86.html)、[雲的故事](http://ast.nhps.tp.edu.tw/home/sally/Source/Unit4-2/Weather/cloud.htm)、wikipedia

氣象雷達(Radar)的主要目的為提供即時的、大範圍的降雨估計，透過發射S-band(波長~10cm)或是C-band(波長~5cm)的脈衝電磁波，偵測周遭空間水氣含量分布得回波強度dBZ後，進行Z-R關係式做簡單降雨估計或使用雙偏極化參數作更精確之估計。

給定空間中雨滴粒徑分布關係(Raindrop size distribution)，透過電磁波散射矩陣計算(T-matrix)，則可得雷達觀測所使用的Z-R關係式，然而問題是，不同的降雨系統如層狀降水(stratiform)與對流降水(convective)的雨滴粒徑分布有所差異，若使用同一組Z-R關係式進行降雨估計，則有高估或低估問題，因此如何掌握當下降雨系統的雨滴粒徑分布，即時修正雷達降雨估計計算，則是另一個課題，雨滴粒徑分布觀測資料是透過一種叫做雨滴譜儀(Parsivel)的觀測儀器得到。


參考資料：
[Raindrop size distribution](https://en.wikipedia.org/wiki/Raindrop_size_distribution)、
[pytmatrix](https://github.com/jleinonen/pytmatrix)、
[雨滴譜儀Parsivel2](https://www.ott.com/products/meteorological-sensors-26/ott-parsivel2-laser-weather-sensor-2392/)


## 使用wradlib套件繪製雷達訊號發射路徑與地形遮蔽

wradlib是一款可讀取雷達資料的python套件，與另一款常用的雷達資料處理套件pyart不同點之一在於，前者擅長讀取德國Selex雷達的rb5資料，後者則是美國NEXRAD雷達資料。

本文參考自wradlilb官網文章[Beam Blockage Calculation using a DEM](https://docs.wradlib.org/en/stable/notebooks/beamblockage/wradlib_beamblock.html)，給定雷達站經緯度與高度資料，結合數值地形模型資料(Digital Elevation Model)，就可以畫出雷達訊號發射路徑及其受地形遮蔽情況(Beam Blockage)。

[完整程式碼](https://github.com/wangjb/colab/blob/master/radar_scan_strategy.ipynb)

## 使用資料

* 臺灣雷達氣象站地理位置(googlemap查詢，僅供參考)

* GMTED2010 dataset - 10N120E_20101117_gmted_mea300.tif

如前所述，為了繪製雷達發射傳播路徑與受地形遮蔽情況，需要雷達經緯度與高度資訊與數值地形模型DEM，前者網路上都能查到，或是googlemap就可估計，[DEM可從美國地質調查局USGS下載](https://topotools.cr.usgs.gov/gmted_viewer/viewer.htm)，這裡使用30角秒資料(解析度25公尺~42公尺)。必須注意的是，地形資料並沒有包含建築物高度資訊。

## 繪製結果

### S-band雷達，RCWF五分山雷達站 

lon, lat, height = (121.782018, 25.071787, 766.0)

這裡僅顯示掃描範圍一百公里以內的雷達發射路徑與地形遮蔽資料，增加程式變數bins可增加掃描範圍。

從Beam Blockage Fraction可看到，五分山雷達很好的涵蓋了北北基桃和宜蘭地區一定的低空區域，然而較高設站高度(766公尺)雖然換取了更多有效掃描範圍，但也犧牲了較多1000公尺以下的觀測區域(僅於20km以內有效，且高於766公尺)。特別是宜蘭地區受地形遮蔽影響，較無法有效觀測蘭陽溪河谷中上游區域對流發展狀況，想必這也是為什麼氣象局希望在宜蘭設置降雨雷達了吧。

新聞 [宜蘭居民反對設降雨監測雷達 氣象局：有在找備選地點](https://udn.com/news/story/7314/4417726)

參考資料 ： [終止七星山氣象雷達站的設置](https://npda.cpami.gov.tw/tab8/web8_page2.php?park=C)

{% include image.html url="/assets/img/RCWF_1.png" description="五分山雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

站在地球外來看，則會有下面這張圖，紅色線為水平線，綠色線為考慮電磁波被大氣折射所得水平發射路徑，藍色為雷達0.5度發射路徑，虛藍線為main lobe的3dB線，雷達主要發射能量都集中在main lobe上，上圖中黑線BBF為main lobe 3dB範圍內隨距離被地形遮蔽的比例。

{% include image.html url="/assets/img/RCWF_2.png" description="同前圖，考慮地球曲率後。" %}


### S-band雷達，RCHL 花蓮雷達站

lon, lat, height =  (121.628372, 23.988536, 61.0)

西邊陡峭的中央山脈以及南南西方突兀的海岸山脈，使得花蓮雷達站肩負的主要任務不是觀測臺灣本島，而是東方太平洋海面的颱風動態，以爭取更早的預警時間。

{% include image.html url="/assets/img/RCHL_1.png" description="花蓮雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

[他曾協助興建氣象雷達站　50年後Mr. Hal Bogin來台重溫過往點滴](https://www.upmedia.mg/news_info.php?SerialNo=60304)


### S-band雷達，RCKT 墾丁雷達站 

lon, lat, height = (120.855609, 21.900181, 40.85)

從0.5度Beam Blockage Fraction來看，墾丁雷達站扮演的腳色與花蓮站相同，主要提供巴士海峽上的颱風動態。

{% include image.html url="/assets/img/RCKT_1.png" description="墾丁雷達站0.5度仰角發射路徑與所經地形遮蔽情形。" %}

新聞 [遊墾丁一定看過神秘「哈密瓜」！真實作用竟然是…](https://udn.com/news/story/120913/2313449)

### S-band雷達，RCCG 七股雷達站 

lon, lat, height = (120.088978, 23.147402, 33.0)

由於七股居民抗爭，七股雷達站後續會牽涉到現址西北方海堤處。從Beam Blockage Fraction來看，七股雷達站為臺灣西半部提供了絕佳的地面天氣觀測，有效涵蓋從地面以上之觀測區域，對臺灣夏季西南氣流的降雨估計至關重要。

{% include image.html url="/assets/img/RCCG_1.png" description="七股雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

新聞 [台南七股氣象雷達新站動土 可望成新地標](https://www.cna.com.tw/news/ahel/201907240171.aspx)

### C-band雷達，RCSL 新北樹林雷達站 

lon, lat, height = (121.400546, 25.003839, 290.0)

前面提到的4座S-band雷達是臺灣早先建立的雷達觀測站，主要提供大範圍降雨觀測，然而近幾年來短時強降雨等極端降雨致災事件發生頻率越來越高，後續政府針對都會地區防洪需求設置掃描頻率較高(每2分鐘)的C-band雷達，由於波長較短，對強降水區域更為敏感，但反過來說，有效觀測範圍也較小(150公里)，分別於2017、2018、2019年啟用高雄林園、台中南屯、新北樹林防災降雨雷達。

參考資料：[RADAR POLARIMETRY AT S, C, AND X BANDS](https://ams.confex.com/ams/pdfpapers/95684.pdf)

若將樹林雷達站與五分山雷達站的0.5度Beam Blockage Fraction相比，乍看五分山雷達有更好的有效涵蓋範圍，但如前所述，這是由於五分山雷達站蓋在七百多公尺的山上，在有效低空觀測區域來比，高度三百公尺左右的樹林雷達站明顯更勝一籌，在此認知下去看樹林雷達站的Beam Blockage Fraction圖，就會了解確實是非常優秀的選址地點，能有效涵蓋雙北最低三百公尺以上低空區域，東北方達基隆，西南方可監測桃園新竹都會區。

{% include image.html url="/assets/img/RCSL_1.png" description="新北樹林雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

新聞 [全台第3座降雨雷達於新北啟用！防災、淹水預警更精準](https://newtalk.tw/news/view/2019-12-27/346532)

### C-band雷達，RCNT 台中南屯雷達站

lon, lat, height = (120.579441, 24.144220, 293.0)

坐落於大肚山台地西南側望高寮的台中南屯雷達站主要肩負台中都會區防洪重任，觀測區域為東側市區上空與東方雪山山脈區域，南南東方可觀草屯、南投、集集等地上空，東北方可觀卓蘭、石岡、東勢山區降雨，能夠為大安溪、大甲溪、大肚溪、濁水溪中上游集水區提供更密集的降雨估計資料，為台中都會區及下游流域鄉鎮爭取防洪預警時間。

{% include image.html url="/assets/img/RCNT_1.png" description="臺中南屯雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

新聞 [第二座防災降雨雷達　台中望高寮啟用](https://www.setn.com/News.aspx?NewsID=396956)

### C-band雷達，RCLY 高雄林園雷達站 

lon, lat, height = (120.380659, 22.526775, 153.0)

位於軍方林園營區的高雄林園雷達，則可監控高雄都會區以及高屏溪上游荖濃溪與隘寮溪之降雨情況。

{% include image.html url="/assets/img/RCLY_1.png" description="高雄林園雷達站0.5度發射仰角路徑與所經地形遮蔽情形。" %}

新聞 [防豪雨 氣象局南部防災降雨雷達啟用](https://www.chinatimes.com/realtimenews/20170912002898-260405?chdtv)

回到一開始的問題，「雷達能看到多低的雲?」看到這裡想必已經能大概了解台灣目前七座氣象雷達能夠看到多低的雲了，取決於雷達高度與發射仰角，還有雷達觀測波段對水氣的敏感程度也會影響。

那雷達可以看到多高的雲呢? 為什麼有時候衛星雲圖看的到雲，雷達觀測圖卻沒看到呢? 這又是另一個故事了...

enjoy :)