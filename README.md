# ocrtext用于屏幕截图的OCR  

## 介绍
主要参考开源项目以及训练模型： 

[Tr](https://github.com/myhub/tr)

[TrWebOCR](https://github.com/alisen39/TrWebOCR) 

[chineseocr_lite](https://github.com/DayBreak-u/chineseocr_lite)

此项目主要用于非扭曲、倾斜、变形的标准字体。 

增加了手动标注文本框、按颜色查找文字、文字识别白名单等功能。 
针对于类似屏幕截图等应用场景，准确、高效。 
如果使用的非标准字体，或者有扭曲变形等应用，请参考其他OCR项目。 

## 特性
* 中英文识别  
* 手动标注文本框  
* 按颜色查找文字
* 文字识别白名单
* lua可调用的测试文档

* 并发请求  
由于模型本身不支持并发，但通过tornado多进程的方式，能支持一定数量的并发请求。具体并发数取决于机器的配置。


## 安装需求  

### 运行平台  
* ✔ Python 3.6+  
* ✔ Ubuntu 16.04
* ✔ ️Ubuntu 18.04
* ✔ CentOS 7   
* ✔ Docker   

Windows和MacOS系统下推荐通过构建Docker镜像来使用，其他Linux平台暂未测试，可自行安装测试  

### 最低配置要求  
* CPU:    1核  
* 内存:    2G  
* SWAP:   2G  

## 安装说明  
### 服务器部署
1. 安装python3.7  
    推荐使用miniconda
    
2. 安装依赖包  
``` shell script
pip install -r requirements.txt
```

3. 运行  
项目默认运行在`8099`端口：  
``` shell script
python backend/main.py
```

安装成功成功后可直接浏览WEB界面测试： 
web界面入口: http://192.168.31.139:8099 
这里用本机`IP`和端口替代，也可以用localhost测试。


### Docker部署  
使用 Dockerfile 构建 或者直接 Pull镜像  
```shell script
# dockerfile 构建
docker build -t ocrtext:v1 .

# 运行镜像
docker run -itd -p 8099:8099 -v $(pwd):/ocrtext --name ocrtext ocrtext:v1 
```

这里把容器的8089端口映射到了物理机的8089上，但如果你不喜欢映射，去掉run后面的-p 8089:8089 也可以使用docker的IP加8089来访问

## 接口

### 文字识别

**描述：** 进行文字识别与检测的接口

**地址：** /api/tr-run/

**方法：** POST

**请求参数：**

| 参数名称  | 是否必选 | 数据类型 | 描述                                                         |
| --------- | -------- | -------- | :----------------------------------------------------------- |
| file      | 是       | file     | 通过上传的方式来发送图片的字段                               |
| compress  | 否       | int      | 值为空时，默认将图片最短边压缩到960px。非空时，将最短边压缩到该值的大小。 |
| whitelist | 否       | string   | 只返回白名单字符串中包含的字符。                             |
| boxlist   | 否       | list     | 值为空时，自动查找文本框。非空时，按照手动给出的文本框识别文字。格式为：[[x1,y1,x2,y2],[x3,y3,x4,y4]]，注意所包含的每一条范围（如：x1,y1,x2,y2）都只能识别单行或者单列的文字，多行是无法识别的，多行文字就标注多个范围。 |
| colorlist | 否       | list     | 给出多组颜色，相近的颜色为一个列表，预处理后才进行识别。格式为：[["0x000000","0x000001"],["0x111110","0x111111"]]，每个元素是一个字符串，内容是一个16进制的RGB颜色信息。 |
| is_draw   | 否       | bool     | 是否返回，画出文字区域的图片base64值                         |

**返回参数：**

| 参数名称     | 是否必选 | 数据类型 | 描述                                              |
| ------------ | -------- | -------- | ------------------------------------------------- |
| code         | 是       | int      | 识别结果的状态码，识别成功为200，有异常为 400     |
| msg          | 是       | string   | 是否成功识别的描述信息                            |
| words        | 否       | list     | 识别结果，若识别异常则没有此字段                  |
| img_detected | 否       | string   | 画出文字区域的图片base64值，需要请求时发送is_draw |
| speed_time   | 是       | float    | 识别的耗时                                        |

**返回示例：**

```json
{"code": 200, "msg": "\u6210\u529f", "words": [["\u4f60", [519, 66, 621, 136]], ["\u5927\u6742\u8d27\u5e97\u8001\u677f\u4f17", [302, 251, 418, 275]], ["\u8981\u4f60\u547d\u5317\uff01", [306, 270, 408, 296]], ["\u8d3c\u738b\u7687\u752b\u65e5", [390, 325, 475, 350]], ["\u53cc\u5178\u7f51", [209, 423, 310, 452]], ["JU5+\u65b0\u665a", [236, 445, 333, 470]]], "speed_time": 0.46}
```


