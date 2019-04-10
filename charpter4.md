# charpter 4 spring cloud Feign 的使用扩展

## 4.2 Feign的基础功能

### 4.2.2 Feign开启GZI压缩

###  4.3.3  Feign的文件上传

-  feign-form

- 1 编写Feign文件上传客户端:  ch4- 4/ ch4- 4- feign- upload- client/ src/ main/ java/ cn/ springcloud/ book/ feign/ service/ FileUploadFeignService. java

- ```java
  @FeignClient(value = "feign-file-server", configuration = "FeignMultipartSupportConfig.class")
  public interface FileUploadFileService {
    
    /**
  		1. produces, consumes 必填
  		2. 注意区分RequestPart和RequestParam
    **/
    @RequestMapping(method = RequestMethod.POST, value="/uploadFile/server",
                   produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
                   consumes = MediaType.MULTIPART_FROM_DATA_VALUE)
  	public String fileUpload(@RequestPart(value = "file")  MultipartFile file);
  }
  ```

- 2 编写Feign文件上传服务端: ch4- 4/ ch4- 4- feign- file- server/ src/ main/ java/ cn/ springcloud/ book/ feign/ controller/ FeignUploadController. java

```java
@RestController
public class FeignUploadController {
  @PostMapping(value = "/uploadFile/server", consumes = "MediaType.MULTIPART_FROM_DATA_VALUE")
  public String fileUploadServer (MulyipartFile file) throw Exception {
    return file.getOriginalFilename();
  }
} 
```

- ≠–=														 上传文件
  - 依次启动：eureka-server  feign-file-server feign-upload-client 三个应用
  - 访问http://localhost:8011/swagger-ui.html
  - 





