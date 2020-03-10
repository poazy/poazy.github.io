---
layout: post
title:  "tesseract-ocr for windows install"
date:   2020-03-10 23:30:00
categories: Tesseract
tags: Tesseract OCR
author: poazy
---

* content
{:toc}
> 记录一下 Windows 下 tesseract-ocr 安装
>
> 下载、安装、环境变量配置、简易使用



# tesseract-ocr 相关地址

```html
https://github.com/tesseract-ocr
https://github.com/tesseract-ocr/tesseract/wiki
https://github.com/UB-Mannheim/tesseract/wiki
https://digi.bib.uni-mannheim.de/tesseract/
https://sourceforge.net/projects/vietocr/files/jTessBoxEditor/
```



# tesseract-ocr 安装截图

## 下载

```html
https://github.com/UB-Mannheim/tesseract/wiki
# 选择合适的版本下载
https://digi.bib.uni-mannheim.de/tesseract/
```

![](../images/20200310-tesseract-ocr/01download-page01.png)

![](../images/20200310-tesseract-ocr/02download-page02.png)

## 安装

```bash
tesseract-ocr-w64-setup-v5.0.0.20190623.exe
```

![](../images/20200310-tesseract-ocr/03install01.png)

* 选择安装可以识别的中文包

![](../images/20200310-tesseract-ocr/04install02.png)

![](../images/20200310-tesseract-ocr/05install0301.png)

![](../images/20200310-tesseract-ocr/06install0302.png)

![](../images/20200310-tesseract-ocr/07install04.png)

![](../images/20200310-tesseract-ocr/08install05.png)

* 如果下载一直卡在 Connection ... 时需要配置 hosts

> 通过 https://www.ipaddress.com/ 查域名 raw.githubusercontent.com 的IP地址，然后配置 hosts 文件

```bash
199.232.28.133 raw.githubusercontent.com
```

![](../images/20200310-tesseract-ocr/09install06.png)

![](../images/20200310-tesseract-ocr/10install07.png)

![](../images/20200310-tesseract-ocr/11install08.png)

![](../images/20200310-tesseract-ocr/12install0901.png)

![](../images/20200310-tesseract-ocr/13install0902.png)

## 环境变量

* 设置 `TESSDATA_PREFIX` 变量

```bash
TESSDATA_PREFIX
```

![](../images/20200310-tesseract-ocr/14env01.png)

![](../images/20200310-tesseract-ocr/15env02.png)

## 命令测试

* 设置好后可以输入命令 `tesseract --version` 看看是否OK

```bash
tesseract --version
```

![](../images/20200310-tesseract-ocr/16env03.png)

## 验证码识别

```bash
tesseract d:\test\codeImage.jpg d:\test\codeText --psm 10
```

![](../images/20200310-tesseract-ocr/17img01.png)



# 参考

```http
https://www.cnblogs.com/channel9/p/12227166.html
https://stackoverflow.com/questions/57160892/tesseract-error-warning-invalid-resolution-0-dpi-using-70-instead
```

