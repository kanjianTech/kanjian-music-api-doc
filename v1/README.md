# 看见音乐开放API接口文档

## 背景

* 此文档为看见音乐对外提供接口调用的 API 文档。
* 版本号 : v1

## 词汇说明

| 字段           | 说明                     |
| ------------- | ------------------------ |
| app_key       | 应用公钥(后台管理员生成) |
| app_secret    | 应用私钥(后台管理员生成) |

## HTTP请求包格式规范

1. **URL**
    * 支持两种 HTTP 请求 Method ： GET 和 POST
    * 对于 GET 请求，参数全部带在 URL 查询参数中，类似 key1=val1&key2=val2&... 形式
    * 对于 POST 请求，参数在 POST 请求体中

2. **参数**

    * 公共参数
        * 所有 API 都需要公共参数，否则服务端将会因为参数校验失败而无法调用 API 接口。
    
            | 字段           | 类型   | 必填 | 描述               |
            | -------------- | ------ | ---- | ------------------ |
            | app_key        | String | 是   | 应用公钥           |
            | access_token   | String | 是   | Token      |
            | device_id      | String | 是   | 设备唯一ID         |
            | sig            | String | 是   | 参数签名，对除 sig 外所有参数进行签名计算得到的值 |
    
    * sig签名生成算法
        1. 将除了 sig 以外的所有请求参数的原始值按照参数名的字典序排序
        2. 将排序后的参数键值对用&拼接，即拼成 key1=val1&key2=val2&...
        3. 将第二步骤得到的字符串进行 Base64 编码
        4. 将 app_secret 作为哈希 key 对第三步骤得到的 Base64 编码后的字符串进行 HMAC-SHA1 哈希运算得到字节数组
        5. 对第四步骤得到的字节数组进行 MD5 运算得到 32 位字符串，即为 sig

    * keyword 特殊字符参数拼接说明
        >
            keyword 参数的值可以包含一些特殊字符如 %、+ 等，所以在拼接 URL 时 API 调用方需要对 keyword 的参数值做一次 URL 编码，并且防止某些 HTTP 客户端（比如 Ruby HTTP 客户端）对这些参数值做二次编码。
            举个例子：
            比如 keyword 的原始值为字符串"幸福溜+7"，如果不做任何处理拼接进 URL，比如 http://test.api.kanjian.com/v1/search/album?keyword=幸福溜+7&...，那么服务端默认会做一次 URL 解码，将参数 keyword 的值解析为"幸福 7"，这是不正确的。所以 API 调用方在拼接 URL 时需要对 keyword 这种参数的值做一次 URL 编码。

1. **JSON格式**
    * 响应数据以 JSON 格式输出，HTTP 响应头中的 Content-Type 为固定值 application/json,charset=utf-8 ，字符编码格式为 UTF-8。

2. **错误码定义**

    * 错误码输出格式
        * error_no 错误编号，其中 1xx 系列全部表示通用错误、2xx 系列全部表示 认证 相关错误、5xx 系列表示服务器内部错误。
        * error_code 错误码
        * error_desc 错误描述
        * service 出错服务名称

    * 错误码输出范例
        ~~~~
        {
            "error_no": [错误编号],
            "error_code":[错误码],
            "error_desc": [错误描述],
            "service": [出错的服务名]
        }
        ~~~~

    * 错误码定义

        | error_no | error_code                      | 描述                                 |
        | -------- | ------------------------------- | ------------------------------------ |
        | 100      | REQUEST_PARAM_CHECK_FAILED      | 参数校验失败错误                     |
        | 101      | SIGNATURE_CHECK_FAILED          | 签名验证失败                         |
        | 102      | PERMISSION_VALIDATE_FAILED      | 权限校验失败                         |
        | 103      | APP_VALIDATE_FAILED             | app_key无效                          |
        | 104      | REQUEST_OUT_OF_LIMIT            | 请求次数超过限额                     |
        | 200      | ACCESS_TOKEN_INVALID_OR_EXPIRED | access_token无效或者已过期           |
        | 201      | AUTHORIZATION_GRANT_DENIED      | 用户或授权服务器拒绝授予数据访问权限 |
        | 500      | INTERNAL_SERVICE_ERROR          | 调用开放平台内部服务发生错误         |
        | 501      | THIRDPARTY_SERVICE_ERROR        | 调用第三方服务发生错误               |
        | 502      | UNKNOWN_SERVER_ERROR            | 未知服务器错误                       |

## 身份认证流程

在调用具体的API数据接口前需要先通过接口获取access_token访问令牌。
具体流程如下

1. 客户端向看见音乐授权服务器进行身份认证，并要求一个访问令牌。请求示例如下：
    ~~~~
    curl -d "app_key=[app_key]&timestamp=1453116822556&sig=[签名]" http://test.api.kanjian.com/token
    ~~~~
    
    * 参数说明：
        * app_key：即app_key，应用公钥
        * timestamp：Unix毫秒数时间戳，每次请求都要重新生成
        * sig：签名参数

2. 看见音乐服务端确认无误后，向客户端返回access_token访问令牌，以json形式返回如下：
    ~~~~
    {
        "access_token": "b994eebe844fcfcfaa62ecef523ee507",
        "expires_in": 86400 // access_token过期时间，单位为秒
    }
    ~~~~

## API 数据接口

1. 音乐风格

    * 获取风格
        * URL /v1/genre
        * HTTP METHOD GET
        * 参数 公共参数
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | genres         | Array  | 是   | 风格列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 风格ID                 |
            | genre_name     | String | 是   | 风格名称               |
            | genre_desc     | String | 是   | 风格描述               |
            | pic_url        | String | 是   | 风格图                 |

        * 示例
        
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "genres": [
                    {
                        "id": 0,
                        "genre_name": "全部",
                        "genre_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "genre_name": "流行",
                        "genre_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 2,
                        "genre_name": "摇滚",
                        "genre_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    ...
                ]
            }
            ~~~~

    * 通过风格获取专辑
        * URL /v1/genre/\<genre_id\>/album
        * HTTP METHOD GET
        * 参数 公共参数 + genre_id
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | albums         | Array  | 是   | 专辑列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_desc     | String | 是   | 专辑描述               |
            | pic_url        | String | 是   | 专辑图                 |

        * 示例
        
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "albums": [
                    {
                        "id": 0,
                        "album_name": "how you love me",
                        "album_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "album_name": "way to go",
                        "album_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    ...
                ]
            }
            ~~~~
        
    * 根据风格获取单曲列表
        * URL /v1/genre/\<genre_id\>/track
        * HTTP METHOD GET
        * 参数 公共参数 + genre_id
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | tracks         | Array  | 是   | 单曲列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 单曲ID                 |
            | track_name     | String | 是   | 单曲名称               |
            | duration       | String | 是   | 时长/秒                |
            | filesize       | String | 是   | 文件大小/KB            |
            | album_id       | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_pic_url  | String | 是   | 专辑图片               |
            | artists        | Array  | 是   | 艺人列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | artist_id      | Int    | 是   | 音乐人ID               |
            | artist_name    | String | 是   | 音乐人名称             |
            | artist_pic_url | String | 是   | 音乐人图片             |

        * 示例
        
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "我的眼泪你的心",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "Gravity Alterstra",
                                "artist_pic_url": "http://a.com/default/artist.jpg"
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "dsadas",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 95067,
                                "artist_name": "Gravity Alterstra",
                                "artist_pic_url": "http://a.com/default/artist.jpg"
                            }
                        ]
                    },
                    ...
                ]
            }
            ~~~~

2. 音乐人

    * 获取音乐人
        * URL /v1/artist
        * HTTP METHOD GET
        * 参数 公共参数
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | artists        | Array  | 是   | 音乐人列表             |

            | 字段            | 类型   | 必填 | 描述                  |
            | --------------  | ------ | ---- | --------------------- |
            | id              | Int    | 是   | 音乐人ID              |
            | artist_name     | String | 是   | 音乐人名称            |
            | artist_desc     | String | 是   | 音乐人描述            |
            | pic_url         | String | 是   | 音乐人图              |

        * 示例
            
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "artists": [
                    {
                        "id": 0,
                        "artist_name": "jayz",
                        "artist_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "artist_name": "ketty",
                        "artist_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 2,
                        "artist_name": "justin bieber",
                        "artist_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                ...
                ]
            }
            ~~~~
        
    * 通过音乐人获取专辑
        * URL /v1/artist/\<artist_id\>/album
        * HTTP METHOD GET
        * 参数 公共参数 + artist_id
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | albums         | Array  | 是   | 专辑列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_desc     | String | 是   | 专辑描述               |
            | pic_url        | String | 是   | 专辑图                 |

        * 示例
            
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "albums": [
                    {
                        "id": 0,
                        "album_name": "how we roll",
                        "album_desc": "",
                        "pic_url": "image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "album_name": "Reasonable Doubt",
                        "album_desc": "",
                        "pic_url": "image.kanjian.com/xxx/xxx.jpg",
                    },
                    ...
                ]
            }
            ~~~~

    * 根据音乐人获取单曲
        * URL /v1/artist/\<artist_id\>/track
        * HTTP METHOD GET
        * 参数 公共参数 + artist_id
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | tracks         | Array  | 是   | 单曲列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 单曲ID                 |
            | track_name     | String | 是   | 单曲名称               |
            | duration       | String | 是   | 时长/秒                |
            | filesize       | String | 是   | 文件大小/KB            |
            | album_id       | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_pic_url  | String | 是   | 专辑图片               |
            | artists        | Array  | 是   | 艺人列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | artist_id      | Int    | 是   | 音乐人ID               |
            | artist_name    | String | 是   | 音乐人名称             |
            | artist_pic_url | String | 是   | 音乐人图片             |

        * 示例
        
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "我的眼泪你的心",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "T",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "别问我是谁",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "T",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    ...
                ]
            }
            ~~~~

3. 专辑

    * 获取专辑
        * URL /v1/album/\<album_id\>
        * HTTP METHOD GET
        * 参数 公共参数 + album_id
        * 返回值
    
            | 字段            | 类型   | 必填 | 描述                   |
            | --------------  | ------ | ---- | ---------------------- |
            | id              | Int    | 是   | 专辑ID                 |
            | album_name      | String | 是   | 专辑名称               |
            | album_desc      | String | 是   | 专辑描述               |
            | pic_url         | String | 是   | 专辑图                 |

        * 示例
        
            ~~~~
            {
                "id": 10,
                "album_name": "how we take",
                "album_desc": "lalalalla zzzzz",
                "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
            }
            ~~~~

    * 根据专辑获取单曲
        * URL /v1/album/\<album_id\>/track
        * HTTP METHOD GET
        * 参数 公共参数 + album_id
        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | tracks         | Array  | 是   | 单曲列表               |
        
            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 单曲ID                 |
            | track_name     | String | 是   | 单曲名称               |
            | duration       | String | 是   | 时长/秒                |
            | filesize       | String | 是   | 文件大小/KB            |
            | album_id       | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_pic_url  | String | 是   | 专辑图片               |
            | artists        | Array  | 是   | 艺人列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | artist_id      | Int    | 是   | 音乐人ID               |
            | artist_name    | String | 是   | 音乐人名称             |

        * 示例
        
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "我的眼泪你的心",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "TT"
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "别问我是谁",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "AA"
                            }
                        ]
                    },
                    ...
                ]
            }
            ~~~~

4. 单曲

    * 获取单曲
        * URL /v1/track/\<track_id\>
        * HTTP METHOD GET
        * 参数 公共参数 + track_id
        * 返回值
    
            | 字段               | 类型   | 必填 | 描述                           |
            | ------------------ | ------ | ---- | ------------------------------ |
            | id                 | Int    | 是   | 单曲ID                         |
            | track_name         | String | 是   | 单曲名称                       |
            | track_file_path    | String | 是   | 单曲文件地址                   |
            | duration           | String | 是   | 时长/秒                        |
            | filesize           | String | 是   | 文件大小/KB                    |
            | album_id           | Int    | 是   | 专辑ID                         |
            | album_name         | String | 是   | 专辑名称                       |
            | album_pic_url      | String | 是   | 专辑图片                       |
            | artists            | Array  | 是   | 艺人列表                       |

            | 字段            | 类型   | 必填 | 描述                  |
            | --------------  | ------ | ---- | --------------------- |
            | artist_id       | Int    | 是   | 音乐人ID              |
            | artist_name     | String | 是   | 音乐人名称            |
            | artist_pic_url  | String | 是   | 音乐人图片            |

        * 注意

            ```diff
            - 线上文件有严格的防盗链设置，文件地址会一段时间后失效，使用缓存的客户请务必注意缓存的设置时间

        * 示例
        
            ~~~~
            {
                "id": 10,
                "track_name": "how we take",
                "track_file_path": "file.kanjian.com/xxx/xxx.wav",
                "duration": 180,
                "filesize": 3096,
                "album_id": 1,
                "album_name": "way to go",
                "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                "artists": [
                    {
                        "artist_id": 1,
                        "artist_name": "TT",
                        "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    }
                ]
            }
            ~~~~

    * 获取单曲试听地址
        * URL /v1/track/\<track_id\>/audition
        * HTTP METHOD GET
        * 参数 公共参数 + track_id
        * 返回值
    
            | 字段               | 类型   | 必填 | 描述                           |
            | ------------------ | ------ | ---- | ------------------------------ |
            | id                 | Int    | 是   | 单曲ID                         |
            | track_name         | String | 是   | 单曲名称                       |
            | audition_file_path | String | 是   | 单曲试听地址(文件获取速度更快) |
            | duration           | String | 是   | 时长/秒                        |
            | filesize           | String | 是   | 文件大小/KB                    |
            | album_id           | Int    | 是   | 专辑ID                         |
            | album_name         | String | 是   | 专辑名称                       |
            | album_pic_url      | String | 是   | 专辑图片                       |
            | artists            | Array  | 是   | 艺人列表                       |

            | 字段            | 类型   | 必填 | 描述                  |
            | --------------  | ------ | ---- | --------------------- |
            | artist_id       | Int    | 是   | 音乐人ID              |
            | artist_name     | String | 是   | 音乐人名称            |
            | artist_pic_url  | String | 是   | 音乐人图片            |

        * 注意

            ```diff
            - 线上文件有严格的防盗链设置，文件地址会一段时间后失效，使用缓存的客户请务必注意缓存的设置时间

        * 示例
        
            ~~~~
            {
                "id": 10,
                "track_name": "how we take",
                "audition_file_path": "file.kanjian.com/xxx/xxx.wav",
                "duration": 180,
                "filesize": 3096,
                "album_id": 1,
                "album_name": "way to go",
                "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                "artists": [
                    {
                        "artist_id": 1,
                        "artist_name": "TT",
                        "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    }
                ]
            }

5. 搜索接口
    * 搜索音乐人
        * URL /v1/search/artist
        * HTTP METHOD GET
        * 参数 公共参数 + 
    
            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | keyword        | String | 是   | 关键词                 |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | artists        | Array  | 是   | 音乐人列表             |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 音乐人ID               |
            | artist_name    | String | 是   | 音乐人名称             |
            | artist_desc    | String | 是   | 音乐人描述             |
            | pic_url        | String | 是   | 音乐人图               |

        * 示例
    
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "artists": [
                    {
                        "id": 0,
                        "artists_name": "赵大宝儿",
                        "artists_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "artists_name": "北京希望无限文化传媒有限公司",
                        "artists_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    ...
                ]
            }
            ~~~~

    * 搜索专辑
        * URL /v1/search/album
        * HTTP METHOD GET
        * 参数 公共参数 + 
    
            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | keyword        | String | 是   | 关键词                 |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | page           | Int    | 是   | 当前页数               |
            | pre_page       | Int    | 是   | 上一页                 |
            | next_page      | Int    | 是   | 下一页                 |
            | count          | Int    | 是   | 当前页面条数           |
            | albums         | Array  | 是   | 专辑列表               |

            | 字段           | 类型   | 必填 | 描述                   |
            | -------------- | ------ | ---- | ---------------------- |
            | id             | Int    | 是   | 专辑ID                 |
            | album_name     | String | 是   | 专辑名称               |
            | album_desc     | String | 是   | 专辑描述               |
            | pic_url        | String | 是   | 专辑图                 |

        * 示例
    
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "albums": [
                    {
                        "id": 0,
                        "album_name": "赵大宝儿",
                        "album_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    {
                        "id": 1,
                        "album_name": "北京希望无限文化传媒有限公司",
                        "album_desc": "",
                        "pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                    },
                    ...
                ]
            }
            ~~~~

    * 搜索单曲
        * URL /v1/search/track
        * HTTP METHOD GET
        * 参数 公共参数 + 
    
            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | keyword        | String | 是   | 关键词                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | page           | Int    | 是   | 当前页数              |
            | pre_page       | Int    | 是   | 上一页                |
            | next_page      | Int    | 是   | 下一页                |
            | count          | Int    | 是   | 当前页面条数          |
            | tracks         | Array  | 是   | 单曲列表              |

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | id             | Int    | 是   | 单曲ID                |
            | track_name     | String | 是   | 单曲名称              |
            | duration       | String | 是   | 时长/秒               |
            | filesize       | String | 是   | 文件大小/KB           |
            | album_id       | Int    | 是   | 专辑ID                |
            | album_name     | String | 是   | 专辑名称              |
            | album_pic_url  | String | 是   | 专辑图片              |
            | artists        | Array  | 是   | 艺人列表              |

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | artist_id      | Int    | 是   | 音乐人ID              |
            | artist_name    | String | 是   | 音乐人名称            |
            | artist_pic_url | String | 是   | 音乐人图片            |

        * 示例
    
            ~~~~
            {
                "page": 10,
                "pre_page": 9,
                "next_page": 11,
                "count": 30,
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "赵大宝儿",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "周杰",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "北京希望无限文化传媒有限公司",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "Jony J",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    ...
                ]
            }
            ~~~~

6. 批量获取单曲信息
    * 获取某个日期后的更新单曲列表
        * URL /v1/batch_track/list
        * 只传after_date则返回after_date到今天，after_date和before_date都有值则返回after_date和before_date之间
        * 如果没有更新数据，则返回空字符串 ''
        * HTTP METHOD GET
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | after_date        | String | 是   | 日期(2018-05-29)                |
            | before_date        | String | 否   | 日期(2018-05-29)                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | 无         | stream-response  | 是   | 单曲id列表(以\n分隔) 
        
        * 示例
            ~~~~
            2436
            2437
            2439
            ...
            ~~~~
    * 批量获取单曲信息
        * URL /v1/batch_track/info
        * HTTP METHOD POST
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | track_id_list        | Array | 是   | 单曲id列表(按返回id列表切片，每次1000个)                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | count          | Int    | 是   | 当前页面条数          |
            | tracks         | Array  | 是   | 单曲列表 |


            | 字段               | 类型   | 必填 | 描述                           |
            | ------------------ | ------ | ---- | ------------------------------ |
            | id                 | Int    | 是   | 单曲ID                         |
            | track_name         | String | 是   | 单曲名称                       |              
            | duration           | String | 是   | 时长/秒                        |
            | album_id           | Int    | 是   | 专辑ID                         |
            | album_name         | String | 是   | 专辑名称                       |
            | album_pic_url      | String | 是   | 专辑图片                       |
            | artists            | Array  | 是   | 艺人列表                       |

            | 字段            | 类型   | 必填 | 描述                  |
            | --------------  | ------ | ---- | --------------------- |
            | artist_id       | Int    | 是   | 音乐人ID              |
            | artist_name     | String | 是   | 音乐人名称            |
            | artist_pic_url  | String | 是   | 音乐人图片            |
        
        * 示例
            ~~~~
            {
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "赵大宝儿",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "周杰",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "北京希望无限文化传媒有限公司",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "Jony J",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    ...
            }
            ~~~~
            
    * 获取单曲播放链接
        * URL /v1/track/<track_id>/url
        * HTTP METHOD GET
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | track_id        | Int | 是   | 单曲id                |
            | user_id        | Int | 是   | 用户id                |
            | is_vip        | Int | 是   | vip标识                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | music_url          | String    | 是   | 单曲播放链接          |
        
        * 示例
            ~~~~
            {
                "music_url":"http://xxx.kanjian.com/xxx/xxx.mp3"
            }
            ~~~~

7. 批量获取单曲信息(秀堂)
    * 获取某个日期后的更新单曲列表
        * URL /v1/xiutang/batch_track/list
        * 只传after_date则返回after_date到今天，after_date和before_date都有值则返回after_date和before_date之间
        * 如果没有更新数据，则返回空字符串 ''
        * HTTP METHOD GET
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | after_date        | String | 是   | 日期(2018-05-29)                |
            | before_date        | String | 否   | 日期(2018-05-29)                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | 无         | stream-response  | 是   | 单曲id列表(以\n分隔) 
        
        * 示例
            ~~~~
            2436
            2437
            2439
            ...
            ~~~~
    * 批量获取单曲信息
        * URL /v1/xiutang/batch_track/info
        * HTTP METHOD POST
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | track_id_list        | Array | 是   | 单曲id列表(按返回id列表切片，每次1000个)                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | count          | Int    | 是   | 当前页面条数          |
            | tracks         | Array  | 是   | 单曲列表 |


            | 字段               | 类型   | 必填 | 描述                           |
            | ------------------ | ------ | ---- | ------------------------------ |
            | id                 | Int    | 是   | 单曲ID                         |
            | track_name         | String | 是   | 单曲名称                       |              
            | duration           | String | 是   | 时长/秒                        |
            | album_id           | Int    | 是   | 专辑ID                         |
            | album_name         | String | 是   | 专辑名称                       |
            | album_pic_url      | String | 是   | 专辑图片                       |
            | artists            | Array  | 是   | 艺人列表                       |

            | 字段            | 类型   | 必填 | 描述                  |
            | --------------  | ------ | ---- | --------------------- |
            | artist_id       | Int    | 是   | 音乐人ID              |
            | artist_name     | String | 是   | 音乐人名称            |
            | artist_pic_url  | String | 是   | 音乐人图片            |
        
        * 示例
            ~~~~
            {
                "tracks": [
                    {
                        "id": 0,
                        "track_name": "赵大宝儿",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "周杰",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    {
                        "id": 1,
                        "track_name": "北京希望无限文化传媒有限公司",
                        "duration": 180,
                        "filesize": 3096,
                        "album_id": 1,
                        "album_name": "way to go",
                        "album_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                        "artists": [
                            {
                                "artist_id": 1,
                                "artist_name": "Jony J",
                                "artist_pic_url": "http://image.kanjian.com/xxx/xxx.jpg",
                            }
                        ]
                    },
                    ...
            }
            ~~~~
            
    * 获取单曲播放链接
        * URL /v1/xiutang/track/<track_id>/url
        * HTTP METHOD GET
        * 参数 公共参数 + 

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | track_id        | Int | 是   | 单曲id                |
            | user_id        | Int | 是   | 用户id                |
            | is_vip        | Int | 是   | vip标识                |

        * 返回值

            | 字段           | 类型   | 必填 | 描述                  |
            | -------------- | ------ | ---- | --------------------- |
            | music_url          | String    | 是   | 单曲播放链接          |
        
        * 示例
            ~~~~
            {
                "music_url":"http://xxx.kanjian.com/xxx/xxx.mp3"
            }
            ~~~~
    

## 图片文件大小切割

* 访问图片文件，加上size参数即可获得任意的切割文件大小

* 示例
    ~~~~
    http://library-test.kanjian.com/default/artist.jpg?x-oss-process=image/resize,h_500,w_500
    ~~~~

## 线上访问文件说明

* 线上文件访问需要在http头中添加 Referer 字段才能进行访问，访问规则是 http://<app_key>.open-api.kanjian.com

* 线上文件有严格的防盗链设置，文件地址会一段时间后失效，请客户务必注意缓存时间的大小

* 示例
    ~~~~
    例如分配的app_key为abc，则通过如下形式才能访问，否则将返回 403 状态码
    curl -H "Referer: http://abc.open-api.kanjian.com" http://library.kanjian.com/xxx/xxx.jpg
    ~~~~
        
