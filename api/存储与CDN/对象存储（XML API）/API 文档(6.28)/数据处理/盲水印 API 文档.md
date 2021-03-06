## 功能描述

盲水印功能基于腾讯云数据万象，是一种全新的水印模式。通过该功能，您可将水印图以不可见的形式添加到原图信息中，并不会对原图质量产生太大影响。当发现图片被盗取后，您可对疑似被盗取的资源进行盲水印提取，验证图片归属。

通过 API 您可以在上传或下载时为图片添加盲水印，也可以对已添加盲水印的图片进行提取盲水印。


## 请求一：添加盲水印

#### 请求示例

**示例一：上传时添加盲水印**

图片上传时添加盲水印的请求包与 [PUT Object](https://cloud.tencent.com/document/product/436/7749) 接口一致，只需在请求包头部增加图片处理参数 Pic-Operations 并使用盲水印参数即可。

```shell
PUT /<ObjectKey> HTTP/1.1
Host: <BucketName-APPID>.cos.<Region>.myqcloud.com
Date: GMT Date
Authorization: Auth String
Pic-Operations: <PicOperations>
```

>?Authorization: Auth String （详情请参见 [请求签名](https://cloud.tencent.com/document/product/436/7778) 文档）。

#### 请求体

Pic-Operations 为 json 格式的字符串，具体参数如下：

| 名称        | 描述                                                         | 类型  | 是否必选 |
| ----------- | ------------------------------------------------------------ | ----- | -------- |
| is_pic_info | 是否返回原图信息。0表示不返回原图信息，1表示返回原图信息，默认为0 | Int   | 否       |
| rules       | 处理规则，一条规则对应一个处理结果（目前最多支持五条规则），不填则不进行图片处理 | Array | 否       |

rules（json 数组）中每一项具体参数如下：

| 参数名称 | 描述                                                         | 类型   | 是否必选 |
| -------- | ------------------------------------------------------------ | ------ | -------- |
| bucket   | 存储结果的目标存储桶名称，格式为：BucketName-APPID，如果不指定的话默认保存到当前存储桶 | String | 否       |
| fileid   | 处理结果的文件路径名称，如以`/`开头，则存入指定文件夹中。否则，存入原图文件存储的相同目录 | String | 是       |
| rule     | 处理参数，可参见图片处理 API。 若按指定样式处理，则以`style/`开头，后加样式名，如样式名为“test”，则 rule 字段为`style/test` | String | 是       |

使用盲水印需在 rule 中添加水印图参数（watermark），相关内容如下：

```shell
watermark/3/type/<type>/image/<imageUrl>/text/<text>/level/<level>
```

#### 参数说明

| 参数  | 描述                                                         | 类型   | 是否必选 |
| ----- | ------------------------------------------------------------ | ------ | -------- |
| type  | 盲水印类型，有效值：1为半盲水印；2为全盲水印；3为文字盲水印    | Int    | 是       |
| image | 盲水印图片地址，需要经过 URL 安全的  Base64 编码。 当 type 为1或2时必填，type 为3时无效。 指定的水印图片必须同时满足如下 3 个条件：<br>1. 盲水印图片与原图片必须位于同一个对象存储桶下；<br>2. URL 需使用数据万象源站域名（不能使用  CDN 加速、COS 源站域名），例如 examplebucket-1250000000.image.myqcloud.com 属于 CDN 加速域名，不能在水印 URL 中使用；<br>3. URL 必须以`http://`开始，不能省略 http 头，也不能填 https 头，例如 examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png，`https://examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png 就是非法的水印 URL。 | String | 否       |
| text  | 盲水印文字，需要经过 URL 安全的 Base64 编码。当 type 为3时必填，type 为1或2时无效。 | String | 否       |
| level | 只对全盲水印（type=2）有效。level  的取值范围为{1,2,3}，默认值为1，level 值越大则图片受影响程度越大、盲水印效果越好。 | Int    | 否       |

#### 响应体

响应体具体数据内容如下：

| 参数名称     | 类型      | 描述     |
| :----------- | :-------- | :------- |
| UploadResult | Container | 原图信息 |

UploadResult 节点内容：

| 参数名称       | 类型      | 描述         |
| :------------- | :-------- | :----------- |
| OriginalInfo   | Container | 原图信息     |
| ProcessResults | Container | 图片处理结果 |

OriginalInfo 节点内容：

| 节点名称  | 类型      | 描述         |
| :-------- | :-------- | :----------- |
| Key       | String    | 原图文件名   |
| Location  | String    | 图片路径     |
| ImageInfo | Container | 原图图片信息 |

ImageInfo 节点内容：

| 节点名称    | 类型   | 描述         |
| :---------- | :----- | :----------- |
| Format      | String | 格式         |
| Width       | Int    | 图片宽度     |
| Height      | Int    | 图片高度     |
| Quality     | Int    | 图片质量     |
| Ave         | String | 图片主色调   |
| Orientation | Int    | 图片旋转角度 |

ProcessResults 节点内容：

| 节点名称 | 类型      | 描述               |
| :------- | :-------- | :----------------- |
| Object   | Container | 每一个图片处理结果 |

Object 节点内容：

| 节点名称 | 类型   | 描述     |
| :------- | :----- | :------- |
| Key      | String | 文件名   |
| Location | String | 图片路径 |
| Format   | String | 图片格式 |
| Width    | Int    | 图片宽度 |
| Height   | Int    | 图片高度 |
| Size     | Int    | 图片大小 |
| Quality  | Int    | 图片质量 |


>?腾讯云 COS 会默认为每个资源生成经过 CDN 的访问链接 access_url，当业务端尚未开通 CDN 时，仍然可以获得该链接，但是无法访问。

## 实际案例

**请求**

```shell
PUT /exampleobject HTTP/1.1
Host: examplebucket-1250000000.cos.ap-chengdu.myqcloud.com
Date: Tue, 03 Apr 2018 09:06:15 GMT
Authorization:XXXXXXXXXXXX
Pic-Operations:
{
    "is_pic_info": 1,
    "rules": [{
        "fileid": "exampleobject",
        "rule": "watermark/3/type/1/image/aHR0cDovL2V4YW1wbGVzLTEyNTEwMDAwMDQucGljc2gubXlxY2xvdWQuY29tL3NodWl5aW4uanBn"
    }]
}
Content-Length: 64

[Object]
```

**响应**

```shell
HTTP/1.1 200 OK
Content-Type: application/xml
Content-Length: 645
Date: Tue, 03 Apr 2018 09:06:16 GMT
Status: 200 OK
x-cos-request-id: NWFjMzQ0MDZfOTBmYTUwXzZkZV8z****

<UploadResult>
<OriginalInfo>
<Key>exampleobject</Key>
<Location>examplebucket-1250000000.cos.ap-chengdu.myqcloud.com/exampleobject</Location>
<ImageInfo>
<Format>JPEG</Format>
<Width>640</Width>
<Height>427</Height>
<Quality>100</Quality>
<Ave>0xa18454</Ave>
<Orientation>1</Orientation>
</ImageInfo>
</OriginalInfo>
<ProcessResults>
<Object>
<Key>exampleobject</Key>
<Location>examplebucket-1250000000.cos.ap-chengdu.myqcloud.com/exampleobject</Location>
<Format>png</Format>
<Width>640</Width>
<Height>427</Height>
<Size>463092</Size>
<Quality>100</Quality>
</Object>
</ProcessResults>
</UploadResult>
```

>!
- 请求包包体的 filecontent 字段为原图的图片内容，包头 Pic-Operations 的 rule 字段中，imageUrl 为水印图片地址。
- 返回包 process_results 字段的 url 为携带盲水印的图片地址。


**示例二：下载时添加盲水印**

图片下载时添加盲水印与添加普通水印操作相同，只需在图片访问链接后使用 watermark 参数即可。相关内容如下：

```shell
watermark/3/type/<type>/image/<imageUrl>/text/<text>
```

#### 参数说明

| 参数  | 类型   | 是否必选 | 描述                                                         |
| :---- | :----- | :------- | :----------------------------------------------------------- |
| type  | Int    | 是       | 盲水印类型，有效值：1为半盲水印；2为全盲水印；3为文字盲水印    |
| image | String | 否       | 盲水印图片地址，需要经过 URL 安全的 Base64 编码。 当 type 为1或2时必填，type 为3时无效。 指定的水印图片必须同时满足如下 3 个条件： <br>1. 盲水印图片与原图片必须位于同一个对象存储桶下； <br>2. URL 需使用数据万象源站域名（不能使用 CDN 加速、COS 源站域名），例如`examplebucket-1250000000.image.myqcloud.com` 属于 CDN 加速域名，不能在水印 URL 中使用； <br>3. URL 必须以`http://`开始，不能省略`http`头，也不能填`https`头，例如`examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png`， `https://examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png`就是非法的水印 URL。 |
| text  | String | 否       | 盲水印文字，需要经过 URL 安全的 Base64 编码。 当 type 为3时必填，type 为1或2时无效。 |

## 实际案例

```shell
http://examplebucket-1250000000.cos.ap-shanghai.myqcloud.com/sample.jpeg?watermark/3/type/3/text/dGVuY2VudCBjbG91ZA==
```

## 请求二：提取盲水印

#### 请求示例

提取盲水印的请求包与 [PUT Object](https://cloud.tencent.com/document/product/436/7749) 接口一致，只需在请求包头部增加图片处理参数 Pic-Operations 并使用提取盲水印参数（watermark/4）即可。

```shell
PUT /<ObjectKey> HTTP/1.1
Host: <BucketName-APPID>.cos.<Region>.myqcloud.com
Date: GMT Date
Authorization:XXXXXXXXXXXX
Pic-Operations: <PicOperations>
```

>?Authorization: Auth String （详情请参见 [请求签名](https://cloud.tencent.com/document/product/436/7778) 文档）。

#### 请求体

Pic-Operations 为 json 格式的字符串，具体参数如下：

| 参数名称    | 类型  | 是否必选 | 描述                                                         |
| :---------- | :---- | :------- | :----------------------------------------------------------- |
| is_pic_info | Int   | 否       | 是否返回原图信息。0表示不返回原图信息，1表示返回原图信息，默认为0 |
| rules       | Array | 否       | 处理规则，一条规则对应一个处理结果（目前最多支持五条规则），不填则不进行图片处理 |

rules（json 数组）中每一项具体参数如下：

| 参数名称 | 类型   | 是否必选 | 描述                                                         |
| :------- | :----- | :------- | :----------------------------------------------------------- |
| bucket   | String | 否       | 存储结果的目标存储桶名称，格式为：BucketName-APPID，如果不指定的话默认保存到当前存储桶 |
| fileid   | String | 是       | 处理结果的文件路径名称，如以`/`开头，则存入指定文件夹中，否则，存入原图文件存储的同目录 |
| rule     | String | 是       | 处理参数，参见数据万象图片处理 API。 若按指定样式处理，则以`style/`开头，后加样式名，如样式名为“test”，则 rule 字段为`style/test` |

提取盲水印需在 rule 中添加水印图参数（watermark），示例格式如下：

```shell
watermark/4/type/<type>/image/<imageUrl>/text/<text>
```

#### 参数说明

| 参数  | 类型   | 是否必选 | 描述                                                         |
| :---- | :----- | :------- | :----------------------------------------------------------- |
| type  | Int    | 是       | 盲水印类型，有效值：1为半盲水印；2为全盲水印；3为文字盲水印，**必须跟添加盲水印时的 type 类型一致**。 |
| image | String | 否       | 图片地址，根据 type 值填写：<br><li>当 type 为1，则 image 必填，且为原图图片地址；<br><li>当 type 为2，则 image 必填，且为水印图地址；<br><li>当 type 为3，则 image 无需填写（无效）。<br>image 需要经过 URL 安全的 Base64 编码，指定的图片必须同时满足如下3个条件：<br>1. 图片与存在水印的图片必须位于同一个对象存储桶下； <br>2. URL 需使用数据万象源站域名（不能使用 CDN 加速、COS 源站域名），例如`examplebucket-1250000000.image.myqcloud.com`属于 CDN 加速域名，不能在水印 URL 中使用； <br>3. URL 必须以`http://`开始，不能省略`http`头，也不能填`https`头，例如`examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png`， `https://examplebucket-1250000000.picsh.myqcloud.com/shuiyin_2.png` 就是非法的水印 URL。 |
| text  | String | 否       | 盲水印文字，需要经过 URL 安全的 Base64 编码。 当 type 为3时必填，type 为1或2时无效。 |

#### 响应体

响应体中的 UploadResult——ProcessResults——Object 字段中新增 WatermarkStatus 字段，当盲水印提取的请求包中 type 参数为2时携带该参数，其他情况不返回该参数。

| 参数            | 类型 | 父节点 | 描述                                                         |
| :-------------- | :--- | :----- | :----------------------------------------------------------- |
| WatermarkStatus | Int  | Object | 当 type 为2时返回该字段，表示提取到全盲水印的可信度。具体为0-100的数字，75分以上表示确定有盲水印，60-75表示疑似有盲水印，60以下可认为未提取到盲水印 |

## 实际案例

**请求**

```shell
PUT /exampleobject1 HTTP/1.1
Host: examplebucket-1250000000.cos.ap-chengdu.myqcloud.com
Date: Tue, 03 Apr 2018 09:06:15 GMT
Authorization:XXXXXXXXXXXX
Pic-Operations:
{
    "is_pic_info": 1,
    "rules": [{
        "fileid": "exampleobject2",
        "rule": "watermark/4/type/1/image/aHR0cDovL2V4YW1wbGVzLTEyNTEwMDAwMDQucGljc2gubXlxY2xvdWQuY29tL2ZpbGVuYW1lLmpwZWc="
    }]
}
Content-Length: 64

[Object]
```

**响应**

```shell
HTTP/1.1 200 OK
Content-Type: application/xml
Content-Length: 645
Date: Tue, 03 Apr 2018 09:06:16 GMT
Status: 200 OK
x-cos-request-id: NWFjMzQ0MDZfOTBmYTUwXzZkZV8z****

<UploadResult>
<OriginalInfo>
<Key>exampleobject1</Key>
<Location>examplebucket-1250000000.cos.ap-chengdu.myqcloud.com/exampleobject1</Location>
<ImageInfo>
<Format>JPEG</Format>
<Width>640</Width>
<Height>427</Height>
<Quality>100</Quality>
<Ave>0xa18454</Ave>
<Orientation>1</Orientation>
</ImageInfo>
</OriginalInfo>
<ProcessResults>
<Object>
<Key>exampleobject2</Key>
<Location>examplebucket-1250000000.cos.ap-chengdu.myqcloud.com/exampleobject2</Location>
<Format>png</Format>
<Width>640</Width>
<Height>427</Height>
<Size>463092</Size>
<Quality>100</Quality>
</Object>
</ProcessResults>
</UploadResult>
```

>!
>
>- 对于 type 值为1的请求，包头 Pic-Operations 的 rule 字段中，imageUrl 为未带盲水印的原图图片地址；对于 type 值为2的请求，包头 Pic-Operations 的 rule 字段中，imageUrl 为水印图的地址。
>- 返回包 ProcessResults 字段的 url 为提取出来的水印图地址。
