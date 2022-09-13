在建立JSON/HTTP Go服务器时，每个处理程序由三个步骤组成。

1. 将请求的JSON主体解压缩为一个结构。

2. 用请求运行该端点的逻辑，以获得一个结果。

3. 将结果打包并写入响应中。

Protobuf lets you define how you want your data structured, compile your protobuf into code in potentially many languages, and then read and write your structured data to and from different data streams.

