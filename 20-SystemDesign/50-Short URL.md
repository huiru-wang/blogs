
短链：较短的URL链接；内容更加精简，不占用其他内容的空间；
- 是URL的一种形式；二维码也是一种形式；
- 与真实URL是映射关系：`1:1`、 `1:n`

# 工作流程
  
1、访问短链，请求短链服务器，携带短链Id；
2、服务端查询短链Id映射的真实URL，响应重定向301，将真实URL放入Location中；
3、客户端重定向完成跳转；

# 功能要点

1. <font color="#de7802">短链域名</font>：一般使用很短小的域名作为短链服务器：sina.lt/
2. <font color="#de7802">短链唯一id</font>：一般使用6位`[0-9][a-z][A-Z]`混合的唯一Id；$62^6$ = 560亿左右；sina.lt/u8DeA7
3. <font color="#de7802">短链服务器后台存储</font>，唯一Id到真实跳转URL的映射关系，并有过期时间，通常1年；
4. 短链可由用户自行生成，依赖高效的<font color="#de7802">唯一Id生成方式</font>；也可以提前生成，进行消耗；

# 短链如何生成

6位`[0-9][a-z][A-Z]`混合即可表示560亿短链，本质是62进制数；

想要不重复，只能递增，可以对十进制递增，再转62进制；

62进制最大：ZZZZZZ；

对应十进制：56800235583；(短链的最大个数)

# 短链如何存储

1、短链 -> 长链 的映射表；short_url可直接作为主键；(可排序)

| short_url | long_url                                 | status | created_at | expired_at |
| --------- | ---------------------------------------- | ------ | ---------- | ---------- |
| u8DeA7    | https://xxxx.com/dddd/aaaa?ware=dwdawdxx | 0      | 2022-11-11 | 2023-11-11 |

2、用户 -> 短链 的映射表；

| id  | user_id    | short_url |
| --- | ---------- | --------- |
| 1   | uid_111313 | u8DeA7    |

# 短链请求接口

1. <font color="#de7802">写请求</font>：生成唯一Id；也可以提前生成，进行消耗；
	- 这里需要从生成的id中随机消耗，防止完全顺序；可预测；

2. <font color="#de7802">读请求</font>：收到短链请求，查询映射，并响应 301；
	- 可以做分布式缓存；
	- 短链存在热点效应，淘汰策略：LFU；保留访问频次高的缓存；