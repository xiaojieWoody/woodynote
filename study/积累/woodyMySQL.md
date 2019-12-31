* 因为外键约束导致删除数据失败

  ```java
  // 先清除外间约束
  String closeForeignKey = "SET FOREIGN_KEY_CHECKS=0";
  sm.execute(closeForeignKey);
  // 删除数据
  // 恢复外键约束
  closeForeignKey = "SET FOREIGN_KEY_CHECKS=1";
  sm.execute(closeForeignKey);
  ```

* ==查出平均分高于60的学生==
  * `select name from (select avg(price) as avgSco, name from books group by name) as avgTab where avgSco > 60`
* ==查询重复记录==
  * `select cardid from t_community_accesscard_info group by cardid having count(*)>1`

* 如何对分页进行优化？

  * `SELECT * FROM big_table order by xx LIMIT 1000000,20`，这条语句会查询出1000020条的所有数据然后丢弃掉前1000000条，为了避免全表扫描的操作，在order by的列上加**索引**就能通过扫描索引来查询
  * 但是这条语句会查询还是会扫描1000020条，还能改进成==`select id from big_table where id >= 1000000 order by xx LIMIT 0,20`==，用ID作为**过滤**条件将不需要查询的数据直接去除

