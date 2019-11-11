# 基本查询

* 模糊查询

  ```java
  // XStu为Entity的类名，不是表名；cName是属性名，不是表字段名
  public interface IStuDao extends JpaSpecificationExecutor<XStu>, JpaRepository<XStu, String> {
      @Query("FROM XStu stu WHERE stu.cName LIKE CONCAT('%',:name,'%')")
      List<XStu> findByPathLike(@Param("name") String name);
  }
  ```

* 外部通过自定义SQL查询

  ```java
  @Autowired
  protected EntityManager entityManager;
  
  String outsql = "FROM XDIndictorInfoRelease re WHERE re.pathkey LIKE CONCAT('%',:id,'%')";
  Query query = entityManager.createQuery(outsql);
  query.setParameter("id", vo.getCId());
  List content = query.getResultList();
  ```

  