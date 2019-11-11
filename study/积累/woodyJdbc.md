# MS

# 基础

* 场景：x_user和x_at_session通过username关联，x_at_session中存储用户的登录记录（多条），要查出用户是否在线，即查出用户在x_at_session中最新一条数据

  ```sql
  select xu.c_name         as name,
         ss.c_status       as status,
         ss.c_createtime   as createTime,
         ss.c_last_renewal as lastRenewal,
         ss.c_logouttime   as logouttime,
         ss.c_updatetime   as updateTime,
         ss.c_logouttype   as logoutType
  from x_user xu
           left join (select s.c_username,
                             s.c_status,
                             s.c_createtime,
                             s.c_last_renewal,
                             s.c_logouttime,
                             s.c_updatetime,
                             s.c_logouttype
                      from x_at_session s
                               inner join (select max(s.c_createtime) as a, s.c_username
                                           from x_at_session s
                                           group by c_username) x
                                          on x.c_username = s.c_username and x.a = s.c_createtime) ss
                     on ss.c_username = xu.c_name
  
  where 1=1 and ss.c_status = 0 and xu.c_name like '%l%'
  order by ss.c_createtime desc
  limit 1, 5
  ```

  ```java
  // jdbcTemplate
  // pageNumber、pageSize -> pageNumber * pageSize=startIndex
  // limit startIndex,pageSize
  // and ss.c_status = :status 
  // and xu.c_name LIKE CONCAT('%',:name,'%') 
  // put("status", status); put("name", name)
  
  @Value("${spring.datasource.url}")
  String dbur;
  @Value("${spring.datasource.username}")
  String username;
  @Value("${spring.datasource.password}")
  String passwd;
  @Value("${spring.datasource.driverClassName}")
  String driver;
  @Autowired
  DataSource dataSource;
  
  NamedParameterJdbcTemplate namedParameterJdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
  
  if(!CollectionUtils.isEmpty(params)) {
    String whereCondition = " where 1=1 ";
    if(params.containsKey("name")) {
      whereCondition += " and xu.c_name LIKE CONCAT('%',:name,'%') ";
    }
    sql += whereCondition;
    countSql += whereCondition;
  } else {
    params = new HashMap<String, Object>();
  }
  
  List<Map<String, Object>> contentList = namedParameterJdbcTemplate.queryForList(sql + orderBy + limit, params);
  ```

# 总结

