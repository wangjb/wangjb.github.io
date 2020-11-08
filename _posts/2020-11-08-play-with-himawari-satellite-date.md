---
layout: post
title: 用Satpy套件繪製臺灣真實色衛星雲圖
---

身處AI大航海時代的背景下，資料開放是時勢所趨，如何使用資料的domain knowledge固然重要，但手中沒有資料也全無用武之地。今年10月氣象局聯合其他單位舉辦了[「第一屆臺灣氣象產業論壇」暨「第三屆氣候服務工作坊」](https://www.weatherrisk.com/post/%E6%B0%A3%E8%B1%A1%E7%94%A2%E6%A5%AD%E7%99%BC%E5%B1%95%E5%8F%97%E5%88%B0%E9%87%8D%E8%A6%96)，邀請產官學研討論如何推動臺灣氣象產業，其中關鍵問題莫過於開放氣象資料，可以預見氣象局未來定位會在於提供高品質資料，由民間開發各種氣象資料的客製化衍生應用。目前氣象局有設置[開放資料平台](https://opendata.cwb.gov.tw/)，但所提供資料完整性和資料架接方便度顯然還有很大進步空間。

## 日本Himawari-8衛星資料

但其實可以不用透過開放資料平台就可以取得衛星雲圖原始資料，氣象局使用的日本向日葵8號(Himawari-8)衛星資料，日本政府已經開放在Amazon的[AWS資料平台上](https://registry.opendata.aws/noaa-himawari/)，[資料說明文件](https://www.data.jma.go.jp/mscweb/en/himawari89/space_segment/spsg_ahi.html)非常齊全。

<a href="https://www.data.jma.go.jp/mscweb/en/himawari89/space_segment/img/ahi_obs.png" target="_blank"><img src="https://www.data.jma.go.jp/mscweb/en/himawari89/space_segment/img/ahi_obs_s.png" alt="HW data"></a>
資料來源: www.data.jma.go.jp

Himawari-8是一顆同步衛星，對西太平洋區域做全天候觀測，觀測資料分為FLDK(全景)、Region(日本區域)、Target(像是颱風等特定資訊)和Landmark(輔助衛星導航用)，aws資料平台提供前述三種Level 1資料及其他衍生Level 2資料。資料頻率10分鐘一張FLDK，四張Region，四張Target。

## Colab + Satpy + AWS HW8 data

下面是在Colab平台上使用Python Satpy套件讀取AWS Himawari-8衛星資料的範例，參考自[這篇](https://github.com/pytroll/pytroll-examples/blob/master/satpy/ahi_true_color_pyspectral.ipynb)。

### 在colab上安裝Satpy套件與aws cli環境

> 安裝相關Library

```
!sudo apt-get -qq update
!sudo apt-get -qq install libproj-dev proj-data proj-bin
!sudo apt-get -qq install libgeos-dev
!sudo pip -q install cython
!sudo pip -q install cartopy==0.16
!sudo pip uninstall -y shapely
!sudo pip install shapely --no-binary shapely
!sudo apt-get -qq install python3-grib
!sudo apt-get -qq install build-essential python3-dev python3-numpy libhdf4-dev -y
```

> 安裝Satpy套件與讀取aws資料用aws cli、s3fs套件


```
!sudo pip -q install satpy[all] 
!sudo pip -q install awscli s3fs
```

### 讀取Full Disk衛星資料

> import讀取資料所需function

```python
from satpy import Scene, find_files_and_readers
from satpy.resample import get_area_def
from datetime import datetime
```

> 連線amazon aws資料庫搜尋有效Himawari-8衛星資料

```python
import s3fs, datetime
files = find_files_and_readers(
      base_dir="s3://noaa-himawari8/AHI-L1b-FLDK/2020/11/06/0400/",
      fs=s3fs.S3FileSystem(anon=True),
      reader="ahi_hsd",
      start_time=datetime.datetime(2020, 11, 6, 4, 0),
      end_time=datetime.datetime(2020, 11, 6, 4, 0))
```

> 下載資料

```
!aws s3 cp s3://noaa-himawari8/AHI-L1b-FLDK/2020/11/06/0150/ . --no-sign-request --recursive
```

### 繪製全球真實色衛星雲圖

> 讀取資料

```python
scn = Scene(sensor='ahi', filenames=files)
```

> 設定合成真實色圖

```python
rgbname = 'true_color'
scn.load([rgbname])
```

> 依資料中最小區域做native resample

```python
new_scn = scn.resample(scn.min_area(), resampler='native')
```

> 產出資料

```python
new_scn.save_dataset(rgbname, filename='himawari_ahi_truecolor_{datetime}.png'.format(datetime=scn.start_time.strftime('%Y%m%d%H%M')))
```
> 輸出資料大小大約在幾百MB左右

{% include image.html url="/assets/img/himawari_ahi_truecolor_202011060400_S.png" description="資料時間：2020年11月6日4點0分 UTC" %}


### 繪製東亞真實色衛星雲圖

> 保留前面的`scn`變數，定義東亞區域，再透過resample方法轉換得到東亞區域衛星雲圖

```python
from pyresample.utils import get_area_def
area_id = 'east_asia'
x_size = 1600
y_size = 1600
area_extent = (-2000000, -2000000, 2000000, 2000000)
#  Center of Taiwan 23.9738° N, 120.9797° E
projection = '+proj=laea +lat_0=23.9738 +lon_0=120.9797 +ellps=WGS84'

description = "East Asia"
proj_id = "laea_23.9738_120.9797"
areadef = get_area_def(area_id, description, proj_id, projection,x_size, y_size, area_extent)
```

> 對設定區域做resample

```python
local_scene = scn.resample(areadef)
```

> 產出資料

```python
local_scene.show(rgbname)
```

{% include image.html url="/assets/img/himawari_ahi_truecolor_EastAsia_202011060400.png" description="資料時間：2020年11月6日4點0分 UTC，圖片中心為臺灣地理中心 23.9738° N 120.9797° E，邊距上下左右各2000公里。" %}

### 繪製臺灣真實色衛星雲圖

> 修改邊距為490公里，重新resample可得臺灣區域衛星雲圖

{% include image.html url="/assets/img/himawari_ahi_truecolor_Taiwan_202007120200.png" description="資料時間：2020年7月12日2點0分 UTC，圖片中心為臺灣地理中心 23.9738° N, 120.9797° E，邊距上下左右各490公里。" %}

### 繪製颱風真實色衛星雲圖

> 同樣方法讀取Target資料可繪製颱風衛星雲圖

```python
import s3fs, datetime
files = find_files_and_readers(
      base_dir="s3://noaa-himawari8/AHI-L1b-Target/2020/11/06/0400/",
      fs=s3fs.S3FileSystem(anon=True),
      reader="ahi_hsd",
      start_time=datetime.datetime(2020, 11, 6, 4, 0),
      end_time=datetime.datetime(2020, 11, 6, 4, 0))
```

> 一次掃描週期有四筆資料，選取其中一筆R301資料繪製

```python
strings = ['R301']
files = {'ahi_hsd':  [x for x in files['ahi_hsd'] if any(s in x for s in strings)] }
```

{% include image.html url="/assets/img/himawari_ahi_truecolor_Target1_202011060402.png" description="資料時間：2020年11月6日4點02分 UTC，颱風閃電正經過墾丁外海。" %}

### 繪製日本真實色衛星雲圖

> 同樣方法可得日本真實色衛星雲圖

```python
import s3fs, datetime
files = find_files_and_readers(
      base_dir="s3://noaa-himawari8/AHI-L1b-Japan/2020/08/19/0300/",
      fs=s3fs.S3FileSystem(anon=True),
      reader="ahi_hsd",
      start_time=datetime.datetime(2020, 8, 19, 3, 0),
      end_time=datetime.datetime(2020, 8, 19, 3, 0))
```

```python
strings = ['JP01']
files = {'ahi_hsd':  [x for x in files['ahi_hsd'] if any(s in x for s in strings)] }
```

{% include image.html url="/assets/img/himawari_ahi_truecolor_Japan1_202008190300.png" description="資料時間：2020年8月19日3點0分 UTC" %}


enjoy :)


### 參考資料

[Load Himawari-8 AHI data and generate true color RGB](https://github.com/pytroll/pytroll-examples/blob/master/satpy/ahi_true_color_pyspectral.ipynb)

[Introduction to Himawari-8 RGB composite imagery](https://www.data.jma.go.jp/mscweb/technotes/msctechrep65-1.pdf)