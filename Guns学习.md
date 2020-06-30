#### 使用AOP实现权限校验

1. 自定义一个注解@Permission，作为一个切点，只要有这个注解就要拦截并进行校验

   ```java
   @Inherited
   @Retention(RetentionPolicy.RUNTIME)
   @Target({ElementType.METHOD})
   public @interface Permission {
   
   }
   ```

2. 定义一个切点，这个切点所切入的不是某个类的某些方法，而是被某个注解所修饰的方法

   ```java
   @Pointcut(value = "@annotation(cn.stylefeng.guns.base.auth.annotion.Permission)")
   private void cutPermission() {
   
   }
   
   //使用方法
   @Around("cutPermission()")
   ```

3. 对这个切点进行环绕操作，获取一个JoinPoint对象

4. 校验通过就执行joinPoint.proceed()，也就是被切入的方法的执行体

