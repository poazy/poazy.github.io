---
layout: post
title:  "Polr 安装试用"
date:   2022-01-23 22:55:00
categories: 短链接
tags: 短链接 Polr
author: poazy
---


* content
  {:toc}
> Docker 方式安装短链接 Polr 并试用



```bash
https://hub.docker.com/r/ajanvier/polr
https://gitee.com/52itstyle/short_url?_from=gitee_search
https://docs.polrproject.org/en/latest/developer-guide/api/
```



```bash
docker run -d --name polr-dsmm -p 88:8080 \
    -e "APP_NAME=DSMM Short Links" \
    -e "DB_HOST=192.168.0.79" \
    -e "DB_PORT=3316" \
    -e "DB_DATABASE=polr" \
    -e "DB_USERNAME=xxx" \
    -e "DB_PASSWORD=xxx" \
    -e "APP_PROTOCOL=http://" \
    -e "APP_ADDRESS=192.168.0.95:88" \
    -e "SETTING_PSEUDORANDOM_ENDING=true" \
    -e "SETTING_AUTO_API=true" \
    -e "SETTING_ADV_ANALYTICS=true"
    -e "ADMIN_USERNAME=admin" \
    -e "ADMIN_PASSWORD=admin@123456" \
    ajanvier/polr:2.3.0b
```



```bash
###
POST http://192.168.0.95:88/api/v2/action/shorten
Content-Type: application/json

{
"key": "630f104739d3027143c649a286632b",
"url": "https://www.baidu.com/"
}

###
```



```java
import cn.hutool.core.util.URLUtil;
import cn.hutool.http.HttpUtil;
import com.alibaba.fastjson.JSONObject;
import org.junit.Assert;
import org.junit.Test;

public class PolrShortLinksTests {
    private String apiUrl = "http://192.168.0.95:88/api/v2/action/shorten";
    private String mspKey = "48474ac0769f2e712ad9962cf157b7";

    @Test
    public void createShortLinks() {
        JSONObject body = new JSONObject();
        body.put("key", mspKey);
        body.put("url", "https://www.baidu.com/s?wd=" + URLUtil.encode("双十二活动.xlsx"));
        String shortLinks = HttpUtil.post(apiUrl, body.toJSONString(), 5000);
        System.out.println(shortLinks);
        Assert.assertNotNull(shortLinks);
    }

}
```

