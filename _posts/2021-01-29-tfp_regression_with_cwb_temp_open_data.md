---
layout: post
title: 使用氣象局Open Data測站溫度資料建立機率迴歸模型
---

前陣子在Coursera上學了["Probabilistic Deep Learning with TensorFlow 2"](https://www.coursera.org/learn/probabilistic-deep-learning-with-tensorflow2)這門課，驚嘆Tensorflow Probabilty的威力，能同時估計資料不確定性以及模型不確定性，迫不及待來牛刀小試一下。機率迴歸方法在[這裡](https://www.tensorflow.org/probability/examples/Probabilistic_Layers_Regression)有清楚的實作介紹，本文中相關程式碼亦參考自此。


## 使用資料

氣象局開放資料可透過API取得，授權碼註冊帳號即有，[氣象局opendata使用說明](https://opendata.cwb.gov.tw/devManual/insrtuction)。

這裡擷取局屬有人測站的資料，含有過去三十天測站每小時的氣象觀測資料，如測站溫度、測站相對濕度、測站氣壓等等。

資料格式為json，但礙於自身才疏學淺，找不到可以直接轉換的工具，只好暴力轉成Dataframe後使用，有

* "station" 測站資訊

* "stationObsTimes" 測站逐時觀測氣象資料

* "stationObsStatistics" 測站觀測資料的簡單統計資訊(這裡我只擷取溫度資料)

以板橋測站(466880)為例，就資料畫出過去三十天的溫度、相對濕度、測站氣壓、測站風速

{% include image.html url="/assets/img/station_meteo_past30_data.png" description="板橋測站過去三十日氣象觀測資料。由左至右、從上自下分別為溫度、相對濕度、測站氣壓、風速，小圖左為觀測資料依時作圖，小圖右為將觀測資料依日期、時間作格點圖。" %}


可清楚看到在1月8日寒流侵襲造成溫度驟降至10度以下，以及1月21日溫度回升至30度上下的情況。

若不考慮日期，只考慮溫度在每個小時的觀測值，可了解過去三十天中，小時溫度的分布狀況。

{% include image.html url="/assets/img/station_hourly_temperature.png" description="板橋測站過去三十日溫度觀測資料。小時溫度觀測值。" %}

## 對小時溫度資料做機率迴歸分析

一般迴歸分析只擬合出具MSE最小的趨勢線，沒有估計預測值的不確定性，機率迴歸分析在擬合趨勢線同時亦給出預測值不確定性的估計值，這裡的不確定性指的是來自資料本身(Aleatoric)，而非模型本身(Epistemic)的不確定性。

```python
# 建立模型
encoded_shape = 1
model = tf.keras.Sequential([
  tf.keras.layers.Dense(256, activation='relu'),                           
  tf.keras.layers.Dense(256, activation='relu'),                           
  tf.keras.layers.Dense(256, activation='relu'),
  tf.keras.layers.Dense(tfp.layers.IndependentNormal.params_size(encoded_shape)),
  tfp.layers.IndependentNormal(encoded_shape)
])

# 訓練model時所用的 loss function - negative log-likelihood
nll = lambda y, rv_y: -rv_y.log_prob(y)

# 訓練模型
model.compile(optimizer=tf.optimizers.Adam(learning_rate=1e-4), loss=nll)
model.fit(x, y, epochs=2000, verbose=False)

```

與簡單迴歸分析不同的是，模型的最後一層使用了[Probabilistic Layer](https://www.tensorflow.org/probability/api_docs/python/tfp/layers)，這層輸出的是一維正規機率分布。給定平均值與標準差，就可以決定正規機率分布，而機率層的前一層Dense layer，就是要去參數化平均值與標準差的所需參數。

另外，不同於一般訓練模型使用MSE作為loss function，這裡使用negative log-likelihood計算loss，back propagation來逐步調整模型的機率分布逼近訓練資料的分布。而其實MSE即是假設資料分布為Normal下所得到的約束結果，文章[剖析深度學習 (4)：Sigmoid, Softmax怎麼來？為什麼要用MSE和Cross Entropy？談廣義線性模型](https://www.ycc.idv.tw/deep-dl_4.html)把這個關係講的很清楚。


{% include image.html url="/assets/img/modeled_station_hourly_temperature.png" description="對板橋測站過去三十日溫度觀測資料小時溫度觀測值做機率迴歸分析。藍點為觀測值，紅線平均值，綠線兩個標準差位置。" %}

分析可見在清晨六、七點會是一天最低溫的最可能出現時間，這是由於地表熱能透過紅外線長波輻射逸散所造成(輻射冷卻)，最高溫則發生在下午一點前後。從平均溫度值的斜率變化可粗略估計地表太陽輻射加熱效率、以及輻射冷卻降溫效率。


由於測站溫度會受到當時候大氣系統影響，偶而會出現一些較為極端的觀測值，特別在寒流來臨時更為明顯。

{% include image.html url="/assets/img/filtered_station_hourly_temperature.png" description="板橋測站過去三十日溫度觀測資料。小時溫度觀測值。黑框處為超過小時觀測值兩標準差以外的觀測值。" %}


[完整程式碼](https://github.com/wangjb/MLprojs/blob/master/cl_30days_meteo_manned.ipynb)

enjoy :)