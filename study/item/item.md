# 项目启动报错

* `nested exception is org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Failed to determine a suitable driver class`
  * 查看是否将properties文件打包到target相应目录中
  * 右键将resources目录设置成Resource Root目录



# 编码习惯

* 问题：指标数据之前指存在数据库，后来也要存es，注意有没有删除情况，如果需要删除，则也需要删除es中数据
* ==建表时查看表各字段==

