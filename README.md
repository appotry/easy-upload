# easy-seed

(安装地址)[https://greasyfork.org/zh-CN/scripts/423199-easy-seed-pt%E4%B8%80%E9%94%AE%E8%BD%AC%E7%A7%8D]
## 关于

支持国内外主流PT站的转载种子脚本，尽可能减少不必要的重复工作，让发种更易上手。
## 功能
* 支持国内外不同架构的站点互转，自动填写简介、视频参数等信息
* 一些站点规则要求截图必须为缩略图，增加了原图转缩略图功能
* 外站向内站转载时，需要补充豆瓣简介，增加了根据IMDB获取豆瓣简介的功能
* 可以通过站点的yaml统一配置来进行上传页的内容填写，新增支持站点更容易。详见构建部分的站点配置规则
* 支持部分站点对当前资源的快速检索
  
  
## 注意事项
* 目前对音乐、MV、动漫以及软件书籍的种子转载不支持(分类可能不会自动填写)
* 柠檬的上传页只支持电影、剧集、纪录片和MV类别的转载
* 内站的简介中会有一些跟视频截图无关的图片，虽然做了一些过滤，转载到外站后这些无关的图片可能仍会保留下来，需要手动删除。
* 大部分外站需要完整的MediaInfo，而部分内战的官组都没提供，转载到外站时，需要手动获取MediaInfo
* 由于TTG的图片加载策略，需要等页面完全加载完整后再点击转载到其他站，否则种子信息会获取不完整。
* 由于部分内站上传页的分类填写过于混乱，会有部分种子分类填写不上的问题，欢迎提Issue

## 后续计划
* 增加FL、HDT、KG的支持
* 国内很多站点由于没有账号或者发布权限，欢迎大佬帮忙测试以及提PR
* 快速检索列表改为从yaml配置里获取

## 构建

`npm run build`

## 本地调试
新建用户脚本,然后将`@require`下的文件路径改为项目所在目录。

```// ==UserScript==
// @name         Debug
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @require      https://cdn.bootcss.com/jquery/1.7.1/jquery.min.js
// @match        https://passthepopcorn.me/torrents.php?id=*
// @match        http*://*/details.php?id=*
// @match        https://totheglory.im/t/*
// @match        https://beyond-hd.me/torrents/*
// @match        https://lemonhd.org/upload_*
// @match        https://lemonhd.org/details*
// @match        https://blutopia.xyz/torrents/*
// @match        https://blutopia.xyz/torrents?imdb=*
// @match        https://blutopia.xyz/upload/*
// @match        http*://*/upload*

// @require      file:///Users/USER_NAME/../easy-seed/.cache/easy-seed.user.js
// @grant        GM_addStyle
// @grant        GM_xmlhttpRequest
// ==/UserScript==

(function() {
    'use strict';

    // Your code here...
})();
```

`npm run dev`

## 站点配置规则

在配置文件中，比较复杂的是目标站点的相关配置，以HDHome的配置为例，说明一下具体的配置规则。在进行站点配置时需要遵循以下几个原则。
* 在获取到各个站点的数据后，会对数据按统一格式进行规范。视频的属性分成了`category`、`videoType`、`videoCodec`、`audioCodec`、`source`、 `resolution`。
* 上传页面默认只有分类是多属性混合的。内站的分类经常是category和其他几个视频的属性混合在一起的，比如`电影 Remux`就是category和videoType混合在一起的一个例子。其他属性的话一般都比较明确，可以跟规范数据中定义的属性直接进行匹配。但是也有特殊的情况，category可以直接匹配，但是videoType是多个属性混合在一起，比如BHD。这个时候需要将category和videoType的值进行对调。后续筛选的话都是默认对category进行筛选。
* 筛选的目的是category配合其他属性，将站点分类的唯一值筛选出来，并对下拉选择框进行赋值。所以即便其他属性可以直接匹配，但是为了配合category筛选出唯一的一个值，需要其他属性除了配置自己的唯一值外，还需要将属性对应的category的值也加上。比如HDH的上传表单里，2160P对应的option value是1，需要将这个值放在数组的**第一位**(很重要)。数组的后几位也需要将 分类中的`Movies UHD Blu-ray`和`Movies 2160p`这两个值加入数组内。因为这两个分类的视频分辨率就是2160P。其他属性的配置也是同理，直到可以互相取交集取到唯一的分类为止。
* 如果规范数据中的几个属性在站点内没有对应的表单需要填入，则不需要配置`selector`属性，只需要配置map即可。map里遵循的原则同上一条。
* 每个属性中map下的属性不同的站点可能会有不同的增减，比如BHD中，bluray又被详细分成了BD100、BD66、BD50等，这个时候在map中加入这些项即可。在对应站点target数据处理中也需要做好对应的处理。

```yaml
 HDHome:
    url: 'https://hdhome.org'
    host: hdhome.org
    siteType: NexusPHP
    asSource: false
    asTarget: true
    uploadPath: /upload.php
    searchPath: /torrents.php
    searchKey: search
    searchParam:
      search_area: '{key}'
      sort: '5'
      type: desc
    # 标题
    name: 
      # 对应输入框或下拉选择框的选择器
      selector: '#name' 
    # 副标题  
    subtitle:
      selector: 'input[name="small_descr"]'
    # 简介  
    description:
      selector: '#descr'
    # imdb地址
    imdb:
      selector: 'input[name="url"][type="text"]'
    # 豆瓣地址 没有的站点可以省略 
    douban:
      selector: 'input[name="douban_id"]'
    # 是否匿名发布
    anonymous: 
      selector: 'input[name="uplver"]'
    # 标签checkbox
    tags: 
      chineseAudio: '#tag_gy'
      DIY: '#tag_diy'
      cantoneseAudio: '#tag_yy'
      chineseSubtitle: '#tag_zz'
      HDR: '#tag_hdr10'
      HDR10+: '#tag_hdrm'
      DolbyVision: '#tag_db'
     # 分类，电影剧集等 
    category: 
      selector: '#browsecat'
      map:
        movie:
          - '411'
          - '412'
        tv:
          - '425'
          - '426'
        tvPack:
          - '432'
          - '433'
        documentary:
          - '417'
          - '418'
        concert: '441'
        sport:
          - '442'
          - '443'
        cartoon:
          - '444'
          - '445'
        variety: []
    # 视频编码    
    videoCodec:
      selector: 'select[name="codec_sel"]'
      map:
        h264: '1'
        hevc: '12'
        x264: '1'
        x265: '2'
        h265: '2'
        mpeg2: '4'
        mpeg4:
          - '5'
          - '412'
          - '418'
          - '426'
          - '433'
          - '445'
        vc1: '3'
        xvid: '5'
        dvd: '5'
    # 视频来源     
    source:
      selector: 'select[name="source_sel"]'
      map:
        uhdbluray: '9'
        bluray: '1'
        hdtv: '4'
        dvd: '3'
        web: '7'
        vhs: '8'
        hddvd: '8'
    # 音频编码    
    audioCodec:
      selector: 'select[name="audiocodec_sel"]'
      map:
        aac: '6'
        ac3: '15'
        dd: '15'
        dd+: '15'
        dts: '3'
        truehd: '13'
        lpcm: '14'
        dtshdma: '11'
        atmos: '12'
        dtsx: '17'
     # 视频类型 主要为以下几种类型    
    videoType:
      selector: 'select[name="medium_sel"]'
      map:
        uhdbluray:
          - '10'
          - '499'
        bluray:
          - '1'
          - '450'
        remux:
          - '3'
          - '415'
        encode:
          - '7'
          - '411'
        web:
          - '11'
          - '411'
        hdtv:
          - '5'
          - '412'
          - '413'
        dvd:
          - ''
          - '411'
        dvdrip:
          - '7'
          - '411'
        other: ''
    # 分辨率    
    resolution:
      selector: 'select[name="standard_sel"]'
      map:
        2160p:
          - '1'
          - '499'
          - '416'
        1080p:
          - '2'
          - '414'
        1080i:
          - '3'
          - '424'
        720p:
          - '4'
          - '413'
        576p:
          - '5'
          - '411'
        480p:
          - '5'
          - '411'
```

