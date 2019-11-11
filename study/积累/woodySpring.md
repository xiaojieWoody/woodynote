# MS

* Spring、SpringMVC、SpringBoot之间的区别
  * Spring MVC是Spring的一个模块，一个web框架，提供了一种轻度耦合的方式来开发web应用
  * SpringBoot基于Spring实现自动配置，降低项目搭建的复杂度

# 经验

# 

# Spring

# SpringBoot

## SpringBoot + 导入文件

```html
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
  
<!--application.properties-->
spring.thymeleaf.prefix=classpath:/templates/  
  
<!--resources/templates/index.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>运维管理-文件导入</title>
</head>
<body>
<h1>资源导入</h1>
<form action="import" id="upfile" method="post" enctype="multipart/form-data">
    <input type="file" name="file"/>
    <input type="submit" value="提交">
</form>
</body>
</html>  
```

```java
@GetMapping("/index")
public String index() {
  return "index";
}

@PostMapping("/import")
@ResponseBody
public ResponseResult fileImport(@RequestParam("file")MultipartFile file) {
  String originalFilename = file.getOriginalFilename();
  public ResponseResult resolveResJsonFile(MultipartFile file) {
    try {
      // file文件中内容 解析 成List对象
      String jsonStr = JSON.toJSONString(JSON.parse(file.getBytes()));
      List<ResDataVO> resDataVOS = JSON.parseObject(jsonStr, new TypeReference<List<ResDataVO>>() {
      });
    } catch (IOException e) {
      e.printStackTrace();
    }
  return new ResponseResult(200, "success", "success", originalFilename);
}
```

## List入参

```java
@PostMapping("/list")
@ResponseBody
public ResponseResult testList(@RequestBody List<ResAuthEveVO> list) {
  String str = JSON.toJSONString(list);
  return new ResponseResult(200, "123", "success", str);
}

@Data
public class ResAuthEveVO implements Serializable {
    private static final long serialVersionUID = -5197769504649608422L;
    private Integer sge;
    private Integer bty;
}
```

```java
post http://127.0.0.1:18471//list
Headers Content-Type:application/json
[{"sge":1,"bty":2},{"sge":3,"bty":4}]
```

# SpringCloud