# MS

* Spring、SpringMVC、SpringBoot之间的区别
  * Spring MVC是Spring的一个模块，一个web框架，提供了一种轻度耦合的方式来开发web应用
  * SpringBoot基于Spring实现自动配置，降低项目搭建的复杂度

# 经验



# Spring

# SpringBoot

## 记录日志-aop

```java
// 操作日志
package com.efast.gac.dtp.bean.core;
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface OperationLog {
    // 操作名称，如"下载"
    String value() default "";
    // 被修改的实体的唯一标识,例如:菜单实体的唯一标识为"id"
    String key() default "id";
}
```

```java
@Aspect
@Component
@Slf4j
public class OperationLogAOP {

    @Pointcut(value = "@annotation(com.efast.gac.dtp.bean.core.OperationLog)")
    public void operationLog() {
    }

    @Around("operationLog()")
    public Object before(ProceedingJoinPoint point) throws Throwable {
        // 先执行业务
        Object result = point.proceed();
        try {
            // 记录日志
            handler(point);
        } catch (Exception e) {
            log.error("记录操作日志异常:{}", e);
        }
        return result;
    }

    private void handler(ProceedingJoinPoint point) throws NoSuchMethodException {
        // 获取拦截的方法名
        Signature signature = point.getSignature();
        MethodSignature sig = null;
        if (!(signature instanceof MethodSignature)) {
            throw new IllegalArgumentException("该注解只能用于方法");
        }
        sig = (MethodSignature) signature;
        // 反射获取方法名
        Class<?> clazz = point.getTarget().getClass();
        Method currentMethod = clazz.getMethod(sig.getName(), sig.getParameterTypes());
        // 获取方法上的注解
        OperationLog annotation = currentMethod.getAnnotation(OperationLog.class);
        // 操作名称
        String operation = annotation.value();
        // 获取用户token
        HttpServletRequest request = HttpKit.getRequest();
        // 获取请求参数
        String issuedId = request.getParameter("issuedId");
        if (StringUtils.isNullOrEmpty(issuedId)) {
            log.error("【操作日志】请求参数issuedId为null");
            return;
        }
        String token = request.getHeader("Authorization");
        String username = JwtUtil.getUsername(token);
        Long userId = JwtUtil.getUserId(token);

        // 查询用户
        User user = userService.findById(userId);
        if (user == null) {
            log.error("【操作日志】用户不存在");
            return;
        }
        // 用户角色名称
        String userRoleName = getUserRoleName(user);
        // 插入数据库
    }
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