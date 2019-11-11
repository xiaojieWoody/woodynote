# 文件

* 文件下载

  * 查询数据并以文件形式下载

    ```java
    byte[] bytes = JSON.toJSONBytes(auditArray);
    HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();
    try {
      String fileName = getToDayDateStr() + "_log.txt";
      response.addHeader("Content-Disposition", "attachment;filename=" + fileName);
      response.addHeader("filename", fileName);
      OutputStream toClient = new BufferedOutputStream(response.getOutputStream());
      response.setContentType("application/octetc-download");
      toClient.write(bytes);
      toClient.flush();
      toClient.close();
    } catch (Exception e) {
      e.printStackTrace();
      logger.error("文件下载异常！", e);
      return new ResponseResult(-1, "文件下载异常！", "", null);
    }
    ```

* 文件上传

  * 获取文件内容并存入数据库

    ```java
    //【1】. application.properties
    spring.thymeleaf.prefix=classpath:/templates/
    //【2】. pom.xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    //【3】. resource/templates/目录下index.html
    <form action="import" id="upfile" method="post" enctype="multipart/form-data">
        <input type="file" name="file"/>
        <input type="submit" value="提交">
    </form>
    //【4】. Controller
    @GetMapping("/index")
    public String index() {
      return "index";
    }  
    @PostMapping("/import")
    @ResponseBody
    public ResponseResult fileImport(@RequestParam("file")MultipartFile file) {
      String originalFilename = file.getOriginalFilename();
      return service.resolveResJsonFile(file);
    }
    //【5】. service 获取文件内容并解析成List
    String jsonStr = JSON.toJSONString(JSON.parse(file.getBytes()));
    List<XOpsAudit> resData = JSON.parseObject(jsonStr, new TypeReference<List<XOpsAudit>>() {
    });
    ```

    