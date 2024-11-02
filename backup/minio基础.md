**官网**：[官网](https://min.io)

**文档**：[文档](https://minio-java.min.io)

**快速开始**：[quickStart](https://www.minio.org.cn/docs/minio/linux/developers/java/minio-java.html#minio-java-quickstart)



**安装**

```shell
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
MINIO_ROOT_USER=wwb MINIO_ROOT_PASSWORD=pubgM666 ./minio server /mnt/min_data --console-address ":9001"
```

**access_key and secret_key**

```text
access_key: Um89W6wboxiiVeLUiYBn
secret_key: 4MmSQrEy5t8xpIbACZLmnYhidaOcIEt246GiSPRM
```

**上传文件demo**

```java
package com.minio;

import io.minio.*;

public class Exec {
    public static void main(String[] args) throws Exception{
        MinioClient client = new MinioClient.Builder().endpoint("http://172.24.192.134:9000")
                .credentials("Um89W6wboxiiVeLUiYBn", "4MmSQrEy5t8xpIbACZLmnYhidaOcIEt246GiSPRM")
                .build();

        boolean exists = client.bucketExists(BucketExistsArgs.builder().bucket("test").build());
        if(!exists) {
            client.makeBucket(MakeBucketArgs.builder().bucket("test").build());
        }

        client.uploadObject(UploadObjectArgs.builder()
                .object("RocketMQ api实战.md")
                .filename("C:\\Users\\wwb\\Desktop\\typora_record\\RocketMQ api实战.md")
                .bucket("test")
                .build());

        client.close();
    }
}
```



**minio对象属性：**

- object name：对象的唯一名称

- bucket name：对象所在的存储桶名称

- content type：对象的mime类型，例如：image/jpeg、application//pdf

- content length：对象的大小，以byte为单位

- last modified：对象的最后修改时间

- user-defined metadata：对象的自定义元数据，例如作者、创建日期等

- etag：对象的唯一标识符，通常是一个哈希值

  

**spring-boot对文件上传大小默认大小为1MB，需要覆盖配置**

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 1TB
      max-request-size: 1TB
```



**获取对象链接**

```java
String url = client.getPresignedObjectUrl(GetPresignedObjectUrlArgs
                .builder()
                .bucket("test")
                .object("RocketMQ api实战.md")
                .method(Method.GET)
                .expiry(5, TimeUnit.DAYS)
                .build());
```



**列出文件列表**

```java
/**
 * 测试列出文件列表
 * @throws Exception 异常
 */
static void testListObjs() throws Exception{
    Iterable<Result<Item>> objs = client.listObjects(ListObjectsArgs.builder()
                                             .bucket("test")
                                             .prefix("video/a")
                                             .recursive(true)
                                             .build());

    for (Result<Item> obj : objs) {
        System.out.println(obj.get().objectName());
    }

    client.close();
}
```



**列出所有bucket**

```java
/**
 * 测试列出所有bucket
 * @throws Exception 异常
 */
static void testListBuckets() throws Exception {
    List<Bucket> buckets = client.listBuckets();
    buckets.forEach(b -> System.out.println(b.name()));
    client.close();
}
```



**下载文件**

```java
/**
 * 测试下载文件
 * @throws Exception 异常
 */
static void testGetObj() throws Exception {
    GetObjectResponse resp = client.getObject(GetObjectArgs.builder()
                                      .bucket("test")
                                      .object("9666489052.mp4")
                                      .build());

    byte[] data = resp.readAllBytes();
    FileOutputStream outputStream = new FileOutputStream(resp.object());
    outputStream.write(data);
    outputStream.flush();
    outputStream.close();

    client.close();
}
```



**复制文件**

```java
/**
 * 测试复制文件
 * @throws Exception 异常
 */
static void testCopyObj() throws Exception {
    CopySource src = new CopySource.Builder().bucket("test").object("9666489052.mp4").build();
    client.copyObject(CopyObjectArgs.builder()
                      .source(src)
                      .bucket("learn")
                      .object("美女舞蹈.mp4")
                      .build());

    client.close();
}
```



**删除文件**

```java
/**
 * 测试删除文件
 * @throws Exception 异常
 */
static void testDeleteObj() throws Exception {
    client.removeObject(RemoveObjectArgs.builder()
                        .bucket("learn")
                        .object("美女舞蹈.mp4")
                        .build());

    client.close();
}
```

