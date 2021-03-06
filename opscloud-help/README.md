# OpsCloud使用文档

#### 开发环境

* MacBook MacOS
* JDK8
* IntelliJ IDEA2019.3
* Maven 3
* WebStorm

#### 运行环境

* Windows/Linux/MacOS
* JDK 8
* CPU2/RAM4GB
* Redis 3
* Mysql 8
* Jumpserver 1.5.6
* Zabbix 4.0

```bash
liangjiandeMacBook-Pro:~ liangjian$ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```



![](https://opscloud-store.oss-cn-hangzhou.aliyuncs.com/github/gif/oc-webxterm-2.gif)





![](https://opscloud-store.oss-cn-hangzhou.aliyuncs.com/github/gif/oc-playbook-1.gif)

 

```java
package com.baiyi.opscloud.oc3test;

import com.alibaba.fastjson.JSON;
import com.baiyi.opscloud.BaseUnit;
import com.baiyi.opscloud.common.util.JSONMapper;
import com.fasterxml.jackson.databind.JsonNode;
import com.google.common.base.Joiner;
import lombok.Data;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.methods.HttpPut;
import org.apache.http.client.protocol.HttpClientContext;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.junit.jupiter.api.Test;

import java.io.IOException;

/**
 * @Author baiyi
 * @Date 2020/5/19 3:30 下午
 * @Version 1.0
 */
public class Oc3Test extends BaseUnit {

    private static final String TOKEN_KEY = "x-token";

    // API-TOKEN 在个人详情中申请
    private static final String X_TOKEN = "AAAAAAAAAAA";

    private static final String OC3_URL = "https://oc3.ops.yangege.cn";

    private static final String queryServer = "/oc3/server/page/fuzzy/query";

    private HttpPost getHttpPostClient(String url) {
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(20000)
                .setConnectTimeout(20000)
                .setConnectionRequestTimeout(20000)
                .build();
        HttpPost httpPost = new HttpPost(Joiner.on("").join(OC3_URL, url));
        httpPost.setConfig(requestConfig);
        httpPost.setHeader(TOKEN_KEY, X_TOKEN);
        httpPost.setHeader("Content-Type", "application/json;charset=utf-8");
        return httpPost;
    }

    @Data
    public class PageQueryServerParam {
        private Integer serverGroupId; // 服务器组id
        private Integer envType; // 环境
        private Integer tagId;
        private Integer extend = 0; // 不显示详情
        private Integer page = 1;
        private Integer length = 100;
    }

    @Test
    void testQueryServer() {
        queryServer(null, null, null);
    }

    private void queryServer(Integer serverGroupId, Integer envType, 
    Integer tagId) {
        HttpPost httpPost = getHttpPostClient(queryServer);
        PageQueryServerParam pageQueryServerParam = new PageQueryServerParam();
        pageQueryServerParam.setServerGroupId(serverGroupId);
        pageQueryServerParam.setEnvType(envType);
        pageQueryServerParam.setTagId(tagId);

        HttpClient httpClient = HttpClients.createDefault();
        try {
            httpPost.setEntity(
            new StringEntity(JSON.toJSONString(pageQueryServerParam), "utf-8"));
            HttpResponse response 
            = httpClient.execute(httpPost, new HttpClientContext());

            HttpEntity entity = response.getEntity();
            byte[] data = EntityUtils.toByteArray(entity);
            JsonNode jsonNode = readTree(data);
            // server list
            System.err.println(jsonNode.get("body").get("data"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static JsonNode readTree(byte[] data) {
        try {
            JSONMapper mapper = new JSONMapper();
            return mapper.readTree(new String(data));
        } catch (Exception e) {
            return null;
        }
    }

}

```

