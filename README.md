<!--
 * @Date: 2021-11-18 13:16:56
 * @LastEditors: LIULIJING
 * @LastEditTime: 2021-11-18 13:54:54
-->
# MOD10A1第六版用户指南(中文翻译，非官方)

> MOD10A1-V006-UserGuide

## 1. 详细的数据描述
积雪覆盖的土地通常在可见光波段有很高的反射率，在短波红外波段有很低的反射率。归一化积雪指数(NDSI)揭示了这种差异的大小。该数据集的积雪由从MODIS/Terra Snow cover 5分钟 L2 Swath 500米(MOD10_L2)数据集中选取的每个网格单元的单一、最佳观测值组成。根据太阳高度、离最低点的距离和电池覆盖范围，每个观测都代表了电池表面的最佳传感器视图

### 1.1 格式: HDF-EOS2和JPEG快照
### 1.2 文件命名约定：MOD10A1.A2000055.h15v01.006.2016061160800.hdf
pid.Ayyyyddd.h[].v[].[VERsion].[yyyydddhhmmss].hdf
每个下载数据包含与hdf metadata 头文件中相同的部分信息
### 1.3 大小为3.5 Mb左右
### 1.4 空间覆盖：全球
Daily Terra Orbit Tracks 
https://www.ssec.wisc.edu/datacenter/polar_orbit_tracks/
https://lpdaacaster.cr.usgs.gov/
https://landweb.modaps.eosdis.nasa.gov/cgi-bin/developer/tilemap.cgi

Snow Cover: "Snow cover" refers to the amount of land covered by snow at any given time积雪当量
积雪分数、二值积雪覆盖和空间QA已经不再使用
积雪当量，原始NDSI，screen results？筛除、掩蔽，结果，基本QA和积雪反照率 albedo 是reflectance的积分

#### SDS 1: NDSI_SNOW_COVER

| 取值0-255 | NDSI_SNOW_COVER|
| --- | --- |
| 0 - 100 | NDSI Snow Cover NDSI计算值|
| 200 | 无数据
| 201 | 不决定
| 211 | 夜晚 |
| 237 | 内陆水 |
| 239 | 海洋 |
| 250 | 云 |
| 254 | 探测器饱和 |
| 255 | NoData |

#### SDS 2: NDSI_Basic_QA

| 取值0-255 | NDSI_Basic_QA |
| --- | --- |
|0 | best |
|1 | good |
|2 | OK |
|3 | poor|

#### SDS 3: NDSI_Snow_Cover_Algorithm_Flags_QA

| 取值0-255 | NDSI_Snow_Cover_Algorithm_Flags_QA(筛选特殊情况) |
| --- | --- |
| 0 | 内陆水置位 |
| 1 | 低可见光置位 |
| 2 | 低NDSI置位 |
| 3 | 温度/海拔筛选条件置位——>= 281K, <1300m 置位；>=281K, >=1300不置位 |
| 4 | 短波红外异常高置位——SWIR>0.45,置位；0.25<SWIR<0.45,异常出现的雪,不置位 |
| 5 | 备用 |
| 6 | 备用 |
| 7 | 太阳正射角度置位——>70°表示太高了，亮度太低，不确定性增大，>85°表示夜晚 |

#### SDS 4: NDSI——原始NDSI，但是将之放大至了0-10,000

#### SDS 5: Snow_Albedo_Daily_Tile——积雪反照率

1-100积雪反照率

https://www.epa.gov/climate-indicators/climate-change-indicators-snow-cover

## 2. 软件与工具
### 2.1 数据是通过HTTPS获取的
### 2.2 软件和工具
1. NASA的地球观测系统数据和信息系统(EOSDIS)近实时数据
https://earthdata.nasa.gov/earth-observation-data/near-real-time/rapid-response
2. NASA的Goddard太空飞行中心|MODIS陆地全球浏览图像
https://landweb.modaps.eosdis.nasa.gov/cgi-bin/browse/browseMODIS.cgi

3. HDF-EOS到GeoTiff转换工具(HEG)可以对HDF-EOS对象进行重新格式化、重投影、拼接和制作子数据集
4. HDFView查看HDF文件的软件工具
5. ENVI的插件MCTK，可以用于大量批处理
6. NASA的HDF-EOS官网说明

## 3. 数据采集和处理

### 3.1 任务目标
MODIS是美国宇航局地球观测系统卫星Aqua和Terra卫星上的关键仪器。EOS包括卫星、数据收集系统和支持一系列极地轨道和全球科学家团体提供支持的一系列赤道轨道卫星、低频角卫星提供的对陆地表面、生物圈、固体地球、大气和海洋的长期全球观测。EOS正在提升我们对于地球整体综合系统的认知。MODIS在开发经过验证的、全球的和交互的地球系统模型方面发挥着至关重要的作用。这些模型能够足够准确地预测全球变化。以帮助决策者就如何最好地保护我们的环境做出合理的决定。有关更多信息，请参见：  
+ 美国宇航局的地球观测系统  
+ NASA Terra| EOS旗舰  
+ NASA MODIS| 中低分辨率成像光谱辐射计  
### 3.2 数据采集
MODIS传感器包含一个系统，通过这个系统，来自地球的可见光可以通过一个扫描孔进入一个扫描腔，然后到达扫描镜。双面扫描镜将射入的光线反射到内部望远镜上，然后将光线聚集到四个不同的探测器组件上。在光线到达探测器组件之前，它要通过分束器和光谱过滤器，将光线分成四个宽波长范围。每当一个光子撞击探测器组件时，就会产生一个电子。电子被吸收在电容器中，最终被转移到前置放大器。电子从模拟信号转换为数字数据，并且向下连接到地面接收站。EOS地面系统(EGS)包括将EOS和其他NASA地球科学数据归档、处理和分发给科学和用户社区的设施、网络和系统。
### 3.3 数据处理
MODIS科学团队不断寻求改进用于生成MODIS数据集的算法。只要有新的算法可用，MODIS自适应处理系统(MODAPS)就会重新处理整个MODIS收集的数据——大气、陆地、冰冻圈和海洋数据集——并发布一个新版本。版本6(也称为集合6)是NSIDC提供的MODIS的积雪最新版本。NSIDC强烈建议用户使用最新版本。
有关更多信息，可以参考的资源如下，包括已知问题、生产计划和未来计划：
+ 本指南
+ MODIS雪和海洋全球测绘项目
+ NASA戈达德太空飞信中心|MODIS土地质量评估
+ MODIS陆地团队验证|积雪、海冰状态(MOD10/29)

### 3.4 推导技术和算法

#### 3.4.1 处理步骤

#### 3.4.1.1 积雪

MOD10_L2 积雪覆盖算法通过计算MODIS Level 1B 校准辐射的归一化差分积雪指数(NDSI)(Hall and Riggs, 2011)来检测积雪。然后，利用数据筛选来检出错误的数据，并且标记不确定的积雪。最终的输出包括NDSI Snow Cover加上云、水体位置，以及数据用户感兴趣的其他算法的计算结果。
为了生成每天500米的数据集，网格算法将所有MOD10_L2条带映射到一个中间积雪产品(MOD10GA)，然后用作输入。从这个版本(Version 6)开始，选择当天最佳积雪观测和计算积雪反照率的算法已经被加入MOD10GA生成过程。注意，MOD10GA是一个中间产品，并没有在NSIDC存档。
一旦数据被网格化，选择算法使用几个标准来确定从一个到几个MOD10_L2条带映射到每个网格单元的最佳观测结果。选择这些标准是为了获得探测积雪的最佳地表传感器视图：具体来讲，是观测到最近的当地太阳中午，最近的轨道最低点轨道，并且提供了最大的覆盖单元。MOD10GA生成过程存储选定观测数据的NDSI_Snow_Cover, NDSI_Snow_Cover_Basic_QA以及原始NDSI作为独立子集，然后计算描述性QA数据，然后再将数据和元数据写入MOD10A1。

有关MOD10_L2数据集的更多信息，请参阅MODIS/Terra Snow Cover 5-Min L2 Swath 500m, Version 6的文档。

#### 3.4.1.2 积雪反照率
尽管V6的积雪反照率是在MOD10GA生成过程中计算的，但算法与V5相同。一旦选择了最好的MOD10_L2观测值，就使用MOD09GA可见光和近红外(VNIR)波段计算MOD09GA地表反射率产品中相应像素的雪反照率。土地覆盖类型从MODIS综合土地覆盖产品(MCDLCHKM)中读取，并使用各向异性响应函数校正非森林地区的各向异性散射效应。白雪覆盖的森林被认为是朗伯反射镜。积雪反照率算法在Klein和施特略夫2002年的书中有所描述。关于所有积雪数据集的更多细节可以在算法理论基础文档(ATBD)中找到。

#### 3.4.2 版本历史
#### 3.4.3误差来源
#### 3.5 质量评估

