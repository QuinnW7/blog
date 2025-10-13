+++
date = '2025-10-12T17:55:00+08:00'
draft = true
title = '微信公众号对接元器智能体平台协议'
+++

<!--more-->'

连接方式
微信连接元器使用北极星服务名
测试环境：trpc.amai.agent_platform.wxmp_assistant
预发布环境：trpc.amai.agent_platform.wxmp_assistant
现网环境：trpc.amai.agent_platform.wxmp_assistant

平台鉴权
内网调用，暂时不加鉴权

通用错误返回
返回参数is
属性	类型	必填	说明
errcode	number	是	错误码，0表示调用成功
errmsg	string	否	接口调用错误信息

示例
{
"errcode":0,
"errmsg":"ok"
}

1. 智能体操作
   1.1 创建智能体
   接口路径
   /api/wxmp/assistant/create
   接口说明
   当公众号主在微信公众号后台开启智能回复功能时，微信调用该接口创建智能体
   注意事项
   智能体在创建成功后，将自动拉取公众号文章作为智能体的知识库。公众号文章解析完成后，智能体将根据公众号文章内容回复用户的问题。
   请求参数
   属性	类型	必填	说明
   appid	string	是	开启智能体回复的公众号appid
   name	string	是	智能体名称
   description	string	否	智能体描述
   system_promot	string	否	智能体系统提示词
   返回参数
   属性	类型	必填	说明
   status	number	是	智能体当前的状态
   参数说明
   返回的status的枚举值说明，status>=2，说明智能体已经可以正常工作
   值	说明
   1	智能体正在首次学习已发表的文章
   2	智能体首次学习完成
   3	智能体正在更新文章
   4	智能体更新文章已经完成
   5	智能体正在补录发表的文章
   6	智能体补录文章完成

调用示例
请求数据示例
{
"appid": APPID,
"name": NAME,
"description": DESCRIPTION,
"system_promot": SYSTEM_PROMPT
}

返回数据示例
{
"status": STATUS
}


1.2 更新智能体
接口路径
/api/wxmp/assistant/update
接口说明
当公众号主在微信公众号后台修改智能回复设置时，微信调用该接口修改智能体设置
注意事项
暂无
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
name	string	是	智能体名称
description	string	否	智能体描述
system_promot	string	否	智能体系统提示词
is_allow_pm_data	number	否	0或不传：不做修改；1:打开—打开T+2后生效，2:关闭—立即生效
version	number	否	0或不传：直接发布线上；1:更新草稿态
返回参数
属性	类型	必填	说明
status	number	是	智能体当前的状态
调用示例
请求数据示例
{
"appid": APPID,
"name": NAME,
"description": DESCRIPTION,
"system_promot": SYSTEM_PROMPT
}

返回数据示例
{
"status": STATUS
}


1.3 删除智能体
接口路径
/api/wxmp/assistant/delete
接口说明
当公众号主在微信公众号后台关闭智能回复功能时，微信调用该接口删除智能体
注意事项
无
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
返回参数
属性	类型	必填	说明
status	number	是	智能体当前的状态
调用示例
请求数据示例
{
"appid": APPID
}

返回数据示例
{
"status": STATUS
}


1.4 查询智能体
接口路径
/api/wxmp/assistant/detail
接口说明
当公众号主在微信公众号后台开启智能回复功能后，微信调用该接口查询智能体状态
注意事项
暂无
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
返回参数
属性	类型	必填	说明
name	string	是	智能体名称
description	string	否	智能体描述
system_promot	string	否	智能体系统提示词
status	number	是	智能体当前的状态
knowledges	object[]	否	智能体绑定的知识库信息
knowledges结构
属性	类型	必填	说明
name	string	否	知识库名称（公众号知识库暂时没有名称）
knowledge_id	string	是	知识库id
doc_num	number	是	知识库文档数量
type	number	是	知识库种类: 1 - 公众号知识库，2 - 文件知识库
create_time	number	是	创建时间（秒级unix时间戳）
update_time	number	是	最近更新时间（秒级unix时间戳）


调用示例
请求数据示例
{
"appid": APPID
}

返回数据示例
{
"name": NAME,
"description": DESCRIPTION,
"system_promot": SYSTEM_PROMPT,
"status": STATUS
}

1.5 智能体文章处理完成通知
接口路径
接口名称： airagcomplete
URL：https://{ip}:{port}/innerapi/bizcomm/airagcomplete?appname={appname}
Method：Post
Content-Type：application/json

接口说明
智能体处理完公众号文章后，调用该接口通知微信侧智能体已经就绪
注意事项

请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
event_type	number	是	1: 首次拉取公众号历史文章完毕，没有补录  2.首次拉取公众号历史文章完毕，需要补录 3.补录完毕全部文章 4: 新增群发/发布文章 5:更新群发/发布文章 6:删除群发/发布文 章 7:封禁群发/发布文章 8:公众号文章解封 9 用户新增知识库文件解析情况 10 用户删除知识库文件 11: 知识库文件更新
event_id	string	否	本次文章变更事件的 id。 目前 type 为 1、2、3时该字段无需填写
article_id	string	否	目前 type 为4、5、6、7时该字段需填写
msgidx	number	否	目前 type 为 4、5、6、7时该字段需填写。当msgidx 为 0 时，表示图文消息
count	number	否	处理完成的文章数，type 为  4、5、6、7时该字段有效
file_id	string	否	type 为 9 11 时需要填写。
file_rag_status	number	否	type 为 9 11  时需要填写。0 上传中 1 上传成功 2 上传失败 3 解析中4 解析失败 5 解析成功 6 审核失败 7 解析超时 8 文件已删除 9 文件不支持解析（文章content为空）

knowledge_id	string	否	 type 为 9 11 时需要填写

返回参数
无
调用示例
请求数据示例
{
"item":[
{
"appid": APPID,
"event_type": EVENT_TYPE,
"event_id": EVENT_ID,
"article_id": ARTICLE_ID,
"msgidx": MSGIDX,
"file_cos_id" : FILE_COS_ID,
"file_rag_status" : FILE_RAG_STATUS
}
]
}

返回数据示例
{
"errcode":value,
"errmsg":value
}

1.6 发布智能体
接口路径
/api/wxmp/assistant/publish
接口说明
当公众号主在微信公众号后台修改智能体并调试完成时，点击页面发布按钮，线上生效，将"DraftAssistantId"的配置赋值给"AssistantId"
注意事项
暂无
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
返回参数
无
1.7 重置编辑智能体

接口路径

/api/wxmp/assistant/revert

接口说明

重置编辑智能体，用线上智能体配置覆盖草稿态智能体，然后将配置返回前端展示

注意事项

暂无

请求参数

属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
返回参数
属性	类型	必填	说明
description	string	否	智能体描述
system_prompt	string	否	智能体系统提示词
custom_knowledge_ids	string[]	否	自定义知识库id列表



2.文件知识库操作
2.1 创建知识库
接口路径
/api/wxmp/knowledge/create
接口说明
当公众号主在微信公众号后台创建文件知识库时，微信调用该接口创建知识库
注意事项
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
name	string	是	知识库名称
返回参数

属性	类型	必填	说明
knowledge_id	string	是	知识库id

调用示例
请求数据示例
{
"appid": "APPID",
"name": "NAME"
}

返回数据示例
{
"knowledge_id": "KNOWLEDGE_ID"
}


2.2 更新知识库
接口路径
/api/wxmp/knowledge/update
接口说明
当公众号主在微信公众号后台创建文件知识库时，微信调用该接口创建知识库
注意事项
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
knowledge_id	string	是	知识库id
name	string	是	知识库名称
返回参数
调用示例
请求数据示例
{
"appid": "APPID",
"knowledge_id": "KNOWLEDGE_ID",
"name": "NAME"
}

返回数据示例
{
"errcode":0,
"errmsg":"ok"
}




2.3 删除知识库
接口路径
/api/wxmp/knowledge/delete
接口说明
当公众号主在微信公众号后台删除文件知识库时，微信调用该接口删除知识库
注意事项
删除知识库时，知识库需要解除绑定
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
knowledge_id	string	是	知识库id
返回参数
调用示例
请求数据示例
{
"appid": "APPID",
"knowledge_id": "KNOWLEDGE_ID"
}

返回数据示例
{
"errcode":0,
"errmsg":"ok"
}


2.4 查询知识库文件状态
接口路径
/api/wxmp/knowledge/file/detail
接口说明
微信公众号后台可使用该接口查询文件状态
注意事项
文件不存在时，不返回数据
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
knowledge_id	string	是	知识库id，知识库id不传时表示查询公众号文章状态
file_ids	string[]	否	文件id列表，最多10个
articles	object[]	否	文章id列表，最多10个，查询公众号文章时填写
article结构
属性	类型	必填	说明
article_id	string	否	公众号图文消息id
msgidx	number	否	公众号文章在图文消息中的序号，0表示整个图文消息

返回参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
knowledge_id	string	是	知识库id
files	object[]	是	文件信息列表
file结构
属性	类型	必填	说明
file_id	string	是	文件id
article_id	string	否	公众号图文消息id
msgidx	number	否	公众号文章在图文消息中的序号
file_name	string	是	文件名
file_rag_status	number	是	文件状态： 3 解析中 4 解析失败 5 解析成功  7 解析超时 8 文件已删除 9 文件不支持解析（文章content为空）
file_type	string	是	文件类型： PDF，DOC，DOCX，TXT，HTML

调用示例
请求数据示例
{
"appid": "APPID",
"knowledge_id": "KNOWLEDGE_ID",
"file_ids": ["fileid1", "fileid2"]
}

返回数据示例
{
"appid": "APPID",
"knowledge_id": "KNOWLEDGE_ID",
"files": [
{
"file_id": "fileid1",
"file_name": "filename.pdf",
"file_rag_status": 3,
"file_type": "PDF"
},
{
"file_id": "fileid2",
"file_name": "filename.doc",
"file_rag_status": 3,
"file_type": "DOC"
}
]
}




3.知识库文件(包含公众号文章)相关接口
3.1 接收知识库文件/公众号文章事件通知
接口路径
/api/wxmp/article/event_notity
接口说明
当公众号文章发生变更时，微信调用该接口通知元器更新知识库内容
注意事项
元器在接收到通知后，需要调用微信接口拉取被修改的文章内容
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
event_type	number	是	公众号文章变更事件类型，4: 新增群发/发布文章 5:更新群发/发布文章 6:删除群发/发布文章 7:封禁群发/发布文章  8: 公众号文章解封 9:用户新增知识库文章 10：用户删除知识库文章  11: 知识库文件更新 12: 用户删除知识库
event_id	string 	是	本次文章变更事件的 id
article_id	string	否	type 为 4、5、6、7时填写。公众号图文消息id
msgidx	number	否	type 为 4、5、6、7时填写。公众号文章在图文消息中的序号，0表示整个图文消息
file_id	string 	否	type 为 9、10 、11时填写
file_cos_url	string	否	type 为 9 、11时填写。注意只有 1 天有效期
knowledge_id	string	否	type 为  9、10 、11时填写。文件知识库 id
file_name	string	是	type 为 9 、11时填写。文件名
file_type	string	是	type 为9 、11时填写。文件类型： PDF，DOC，DOCX，TXT
返回参数
无
调用示例
请求数据示例
{
"item":[
{
"appid": APPID,
"event_type": EVENT_TYPE,
"event_id" : EVENT_ID,
"article_id" : ARTICLE_ID,
"msgidx": MSGIDX，
"file_cos_id": FILE_COS_ID,
"file_cos_url": FILE_COS_URL
"knowledge_id": knowledge_id
}
]
}


返回数据示例
暂无

3.2 文章批量拉取
接口路径
接口名称： getbizpublishlist
URL：https://{ip}:{port}/innerapi/bizcomm/getbizpublishlist?appname={appname}
Method：Post
Content-Type：application/json

接口说明
智能体创建成功后，元器调用该接口批量拉取微信公众号文章
注意事项
暂无
请求参数
属性	类型	必填	说明
appid	string	是	微信公众号ID
fetch_type	number	否	 0 按照 offset 和 count 批量拉取，1 表示article_id 和 msgidx拉取单篇文章
offset	number	否	偏移量。0表示从第一次请求，从第二次请求开始将该值赋值为上一次请求拿到的next_offset 值
count	number	否	单次拉取批量文章数，取值范围待定。一次性不要超过 20 篇
filter_pay_subscribe	number	否	1 表示过滤掉付费文章，0 表示也要拉取付费文章，默认为 0
article_id	string	否	图文消息 id
msgidx_list	Array	否	群发消息中具体某篇文章的序号， 当msgidx_list size 为空时，表示拉取整个图文消息

返回参数
参数名	类型	说明
next_offset	number	为0表示全量拉取完毕，否则就把该值填到下次请求的入参 offset 里
item_count	number	本次调用获取的文章消息数量，不保证该值会等于入参 count
article_id	string	图文消息id
title	string	该篇图文的标题
author	string	作者
digest	string	图文消息的摘要，仅有单图文消息才有摘要，多图文此处为空
content	string	图文消息的具体内容，支持HTML标签
content_source_url	string	图文消息的原文地址，即点击“阅读原文”后的URL
thumb_media_id	string	图文消息的封面图片素材id（一定是永久MediaID）
show_cover_pic	number	是否显示封面，0为false，即不显示，1为true，即显示(默认)
need_open_comment	number	Uint32 是否打开评论，0不打开(默认)，1打开
only_fans_can_comment	number	Uint32 是否粉丝才可评论，0所有人可评论(默认)，1粉丝才可评论
url	string	图文消息的URL
is_pay_subscribe	bool	该篇图文是否付费
is_deleted	bool	该篇图文是否被删除
create_time	number	该图文消息的创建时间
update_time	number	该图文消息的最后更新时间（跟用户看到的文章内容更新时间口径不一致）
msgidx	number	文章在图文消息中的序号，从1开始排序
picture_list	repeated object	图文消息中的图片信息
+cdn_url	string	图片url

调用示例
请求数据示例
{
"appid": APPID,
"fetch_type": EVENT_TYPE,
"offset" : OFFSET,
"count":COUNT,
"filter_pay_subscribe" : FILTER_PAY_SUBSCRIBE,
"article_id":ARTICLE_ID,
"msgidx_list":[1,2,3,4]
}

返回数据示例
{
"next_offset": value,
"item_count":value,
"item":[
{
"article_id":value,
"content":{
"create_time":value,
"update_time":value,
"set_send_time":value,
"news_item":[
{
"msgidx":value,
"title":value,
"author":value,
"digest":value,
"content":value,
"content_source_url":value,
"thumb_media_id":value,
"show_cover_pic":value,
"url":value,
"thumb_url":value,
"need_open_comment":value,
"only_fans_can_comment":value,
"is_pay_subscribe":value,
"is_deleted":value
"picture_list": [
{
"cdn_url":value;
}
]
}
]
}
}
],
"errcode":value,
"errmsg":value
}


4.用户聊天消息处理
4.1 接收用户聊天消息推送
接口路径
/api/wxmp/message/notify
接口说明
当公众号主在微信公众号后台开启智能回复功能后，微信调用该接口将用户发送给公众号的消息转发给元器
注意事项
暂无
请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
openid	string	是	发送消息的用户ID
msgid	string	是	用户发送的消息ID
send_time	number	是	消息发送时间
msg_type	string	是	用户发送的消息类型（"text"/"voice"）
text	object	否	msg_type为"text"时，消息体是text
+content	string	是	用户发送的文本消息内容
is_debug	number	否	 0：不是调试器消息 1：是调试器消息
voice	object	否	msg_type为"voice"时，消息体是voice
+media_id	string	否	音频 id。后续用该 id 拉取音频具体内容
+format	number	否	音频格式。0：AMR 1 speex  2 mp3  3 wave 4 silk 5 aac

返回参数
无
调用示例
请求数据示例
{
"appid": APPID,
"openid": OPENID,
"msgid": MSGID,
"send_time": SEND_TIME,
"msg_type" : MSG_TYPE,
"text" : {
"content" : CONTENT
},
"is_debug" : IS_DEBUG
"voice": {
"media_id": media_id,
"format": format
},
}

返回数据示例
无
4.2 回复用户聊天消息
接口路径
接口名称： recvaireply
URL：https://{ip}:{port}/innerapi/bizcomm/recvaireply?appname={appname}
Method：Post
Content-Type：application/json

接口说明
元器回答完用户的消息后，调用该接口将回答内容发送给用户
注意事项
暂无
请求参数
属性	类型	必填	说明
msgid	string	是	用户发送的消息ID
openid	string	是	用户 id
msg_type	string	是	回复内容的消息类型（text）
text	object	否	msg_type为text时，消息体是text
+content	string	是	回复的文本消息内容
is_debug	number	否	0：不是调试器消息 1：是调试器消息

返回参数
暂无
调用示例
请求数据示例
{
"msgid": MSGID,
"openid": OPENID,
"msg_type" : MSG_TYPE,
"text" : {
"content" : CONTENT
}
"is_debug" : IS_DEBUG
}

返回数据示例
{
"errcode":value,
"errmsg":value
}

4.3 拉取历史对话

接口路径
/api/wxmp/message/list
接口说明
当公众号主在微信公众号后台调试时，获取历史消息记录
注意事项
暂无

请求参数
属性	类型	必填	说明
appid	string	是	开启智能体回复的公众号appid
openid	string	是	发送消息的用户ID
is_debug	number	是
返回参数
属性	类型	必填	说明
messages	[]message	是	历史对话
message：
属性	类型	必填	说明
msg_type	string	是	回复内容的消息类型（text）
speaker	string	是	user/assistant
text	object	否	msg_type为text时，消息体是text
+content	string	是	回复的文本消息内容
+index	int	是	排序
+created_at	int	是	创建时间

调用示例
请求数据示例
{
"messages": [
{
"speaker" : "assistant",  
"msg_type": "text",
"text": {
"content": "你好，有什么可以帮你的吗？",
"index": 1,
"created_at": 1672448400
}
},
{
"speaker" : "assistant",
"msg_type": "text",
"text": {
"content": "当然可以，你想了解哪方面的信息？",
"index": 2,
"created_at": 1672448460
}
}
]
}


4.4 回复用户聊天消息V2
接口路径
接口名称： v2recvaireply
URL：https://{ip}:{port}/innerapi/bizcomm/v2recvaireply?appname={appname}
Method：Post
Content-Type：application/json

接口说明
元器回答完用户的消息后，调用该接口将回答内容发送给用户。该接口支持对原来的一个消息进行分段发送。同时优化了引文，把引文放到一个单独的字段中。
注意事项
暂无
请求参数
属性	类型	必填	说明
msgid	string	是	用户发送的消息ID
openid	string	是	用户 id
is_debug	number	否	0：不是调试器消息 1：是调试器消息
msgs	[]object	否	消息列表
其中msgs是一个对象数组，对象msg内容如下：
属性	类型	必填	说明
type	string	是	消息类型。目前支持的有效类型："text"
content	string	是	消息内容
quotation	string	否	消息引文内容

调用示例
请求数据示例
{
"msgid":"1234",
"openid":"12345",
"is_debug": 0,
"msgs":[
{
"type":"text",
"content":"1234",
"quotation":"dddd"
},
{
"type":"text",
"content":"2345",
"quotation":"xxxx"
}
]
}




5.拉取多媒体内容
5.1 拉取音频内容
接口路径
接口名称： fetchmediabyid
URL：https://{ip}:{port}/innerapi/bizcomm/fetchmediabyid?appname={appname}
Method：Post
Content-Type：application/json

接口说明
元器收到音频消息时，会拿到一个mediaid，需要调用该接口拿到音频内容

注意事项
暂无

请求参数
属性	类型	必填	说明
appid	string	是	公众号 appd
media_id	string	是	音频 id

返回参数
属性	类型	必填	说明
errcode	number	是	错误码
errmsg	string	否	错误信息
buffer	string	是	音频内容。目前是 silk 格式的

调用示例
请求数据示例
{
"appid": appid,
"media_id": media_id
}

返回数据示例
{
"errcode":value,
"errmsg":value,
"url":value
}


