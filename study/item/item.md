# 项目启动报错

* `nested exception is org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Failed to determine a suitable driver class`
  * 查看是否将properties文件打包到target相应目录中
  * 右键将resources目录设置成Resource Root目录