---
layout: default
title: Spring Transactional
parent: Spring
grand_parent: Java
nav_order: 14
---

# Spring Transactional
{: .no_toc }

## Here are SpringBoot upgrade experience .
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 事务的理解

对事务的理解：事务是指修改数据库数据时，需要满足所有的操作要满足一致性，要么全都操作，要么全都不操作。

例如，当我买一个商品时，大致的步骤如下：

1.数据库中商品的数量减一

2.我的账户余额减去商品的价格。

当执行到第二步时，若发现我的余额不够支付该商品，本次购买便失败，系统应该恢复原来的商品数量（即需要回滚），这便是一个典型的事务，库存减一和账户消费必须一致，要么一起执行成功，要么都不执行。

REQUIRED

REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
REQUIRES_NEW

事务传播行为：REQUIRES_NEW

(1) 在相同的service内的两个事务方法

如果在同一个service类中定义的两个方法， 内层REQUIRES_NEW并不会开启新的事务，save和update中回滚都会导致整个事务的回滚

```java
public class UserService{


    @Transactional(propagation = Propagation.REQUIRED)
    @Override
    public void save(UserRecord userParam) {
        logger.info("开始执行 save {}, {}", userParam.getId(), userParam.getPhone());
        userMapper.save(userParam);
        try {
            logger.info("save 完成---数据库中的值为 {} 开始 sleep", userParam.getId());
            Thread.sleep(1000);
            this.update(userParam);
			//throw new RuntimeException(); 
			//此处异常会导致两个方法都回滚
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


    @Transactional(propagation = Propagation.REQUIRES_NEW)
    @Override
    public Integer update(UserRecord userParam) {
        logger.info("开始执行 update {}, {}", userParam.getId(), userParam.getPhone());
        int result = userMapper.update(userParam);
        try {
            logger.info("update 完成---数据库中的值为 {} 开始 sleep");
            Thread.sleep(1000 * 10);
            logger.info("sleep结束");
			// throw new RuntimeException(); 
			// 此处异常会导致两个方法都回滚
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

(2) 在不同的service内的两个事务方法

如果在不同的service中定义的两个方法， 内层REQUIRES_NEW会开启新的事务，并且二者独立，事务回滚互不影响。

```java

public class AccountService{

 	@Transactional(propagation = Propagation.REQUIRED)
    @Override
    public void save(AccountRecord accountParam) {
        logger.info("开始执行 save {}, {}", accountParam.getId(), accountParam.getPhone());
        accountMapper.save(accountParam);
        try {
            logger.info("save 完成---数据库中的值为 {}", userParam.getId());
     
            Thread.sleep(1000);

            userParam.setName("xxx");
            userParam.setPhone("xxx");
            userService.save(accountParam);
			//throw new RuntimeException();
            // 如果userService.save 正常执行，此处抛出异常，userService.save事务不受影响，仅 AccountService.save回滚。
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (AppException ae) {
            ae.printStackTrace();
        }
    }

}

public class UserService{

 	@Transactional(propagation = Propagation.REQUIRED)
    @Override
    public void save(AccountRecord accountParam) {
        logger.info("开始执行 save {}, {}", accountParam.getId(), accountParam.getPhone());

        UserRecord userParam = new UserRecord();
        userParam.setName(accountParam.getName());
        userParam.setPhone(accountParam.getPhone());
        userMapper.save(userParam);
        try {
            logger.info("save 完成---数据库中的值为 {}", userParam.getId());

			//throw new RuntimeException();
            // 注意在这抛出异常 会导致异常继续向上传递到 AccountService.save中导致两个事务都回滚
            
            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
            // 只回滚到内层事务的，不会影响到userServive2的save回滚
 
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (AppException ae) {
            ae.printStackTrace();
        }
    }

}

```

**SUPPORTS**

SUPPORTS:如果有事务在运行，当前的方法就在这个事务内运行，否则它可以不运行在事务中

**NOT_SUPPORTED**

NOT_SUPPORTED ：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

**NEVER**

NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

**MANDATORY**

MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。

**NESTED**

NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与REQUIRED类似的操作。拥有多个可以回滚的保存点，内部回滚不会对外部事务产生影响。只对DataSourceTransactionManager有效



##  @Transactional 事务失效整理 

### 场景一 操作了非 public 修饰的方法

@Transactional 应用在非 public 修饰的方法上会导致事务失效。

如果Transactional注解应用在非public修饰的方法上，Transactional将会失效。

在Spring AOP 代理时，如上图所示 `TransactionInterceptor` （事务拦截器）在目标方法执行前后进行拦截，`DynamicAdvisedInterceptor`（CglibAopProxy 的内部类）的 `intercept` 方法或 `JdkDynamicAopProxy` 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource `的 `computeTransactionAttribute` 方法，获取Transactional 注解的事务配置信息:

```java
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable {

    protected TransactionAttribute computeTransactionAttribute(Method method, Class<?> targetClass) {
        if (this.allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
            return null;
        } else {
            Class<?> userClass = ClassUtils.getUserClass(targetClass);
            Method specificMethod = ClassUtils.getMostSpecificMethod(method, userClass);
            specificMethod = BridgeMethodResolver.findBridgedMethod(specificMethod);
            TransactionAttribute txAttr = this.findTransactionAttribute(specificMethod);
            if (txAttr != null) {
                return txAttr;
            } else {
                txAttr = this.findTransactionAttribute(specificMethod.getDeclaringClass());
                if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
                    return txAttr;
                } else {
                    if (specificMethod != method) {
                        txAttr = this.findTransactionAttribute(method);
                        if (txAttr != null) {
                            return txAttr;
                        }

                        txAttr = this.findTransactionAttribute(method.getDeclaringClass());
                        if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
                            return txAttr;
                        }
                    }

                    return null;
                }
            }
        }
    }
}
```
此方法会检查目标方法的修饰符是否为 public，不是 public则不会获取 `@Transactional` 的属性配置信息。

所以使用：protected、private修饰的方法上使用 `@Transactional` 注解，虽然事务无效，但不会有任何报错。
 


### 场景二 propagation 设置错误

@Transactional 注解属性 propagation 设置错误

这种失效是由于配置错误，若是错误的配置以下三种 `propagation`，事务将不会发生回滚。

- `TransactionDefinition.PROPAGATION_SUPPORTS` ： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

- `TransactionDefinition.PROPAGATION_NOT_SUPPORTED` ： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

- `TransactionDefinition.PROPAGATION_NEVER` ： 以非事务方式运行，如果当前存在事务，则抛出异常。

不过我们常用的是：

- `TransactionDefinition.PROPAGATION_REQUIRED` ：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是<mark>默认值</mark>。

- `TransactionDefinition.PROPAGATION_REQUIRES_NEW` ： 新建事务，如果当前存在事务，把当前事务挂起。 


### 场景三 rollbackFor 设置错误

@Transactional 注解属性 rollbackFor 设置错误

rollbackFor 可以指定能够触发事务回滚的异常类型。Spring默认抛出了未检查unchecked异常（继承自 RuntimeException 的异常）或者 Error才回滚事务；其他异常不会触发回滚事务。如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 rollbackFor属性。

![异常](/images/spring/exception.jpg)

如果所有异常都回滚:

```java
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= Exception.class)
```

也可以自定义异常:

```java
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= BuzException.class)
```

示例：
```java
public class XxxService{
   	@Transactional(propagation= Propagation.REQUIRED,rollbackFor= Exception.class)
    public void changeMoney(int oldId, int newId, double money) {
        System.out.println("----update come ----");
        Account oldAcc = accountMapper.getAccountById(oldId);
        Account newAcc = accountMapper.getAccountById(newId);
        oldAcc.setMoney(oldAcc.getMoney() - money);
        newAcc.setMoney(oldAcc.getMoney() + money);

        accountMapper.updateAccountId(oldAcc);

        int i = 10/0 ;

        accountMapper.updateAccountId(newAcc);
        System.out.println("----update over ----");
    }
}
```
Exception 包含了 RuntimeException 所以可以回滚。


### 场景四 try-catch 处理了异常


异常被`try{ }catch(){}` 代码块处理了，会导致`@Transactional`失效。

错误示例：该示例不会回滚。

```java
public class XxxService{

	public void catchMoney(int oldId, int newId, double money) {

        System.out.println("----update come ----");
        Account oldAcc = accountMapper.getAccountById(oldId);
        Account newAcc = accountMapper.getAccountById(newId);
        oldAcc.setMoney(oldAcc.getMoney() - money);
        newAcc.setMoney(oldAcc.getMoney() + money);

        try {

            accountMapper.updateAccountId(oldAcc);

            dosomething(); // 会抛出异常

            accountMapper.updateAccountId(newAcc);
        }catch (Exception e){
            // ....
        }
    }

    public void dosomething() {
        System.out.println("----do something exception ----");
        int x = 10/0 ;
    }
}
```

如果`dosomething`方法内部抛了异常，而 `catchMoney` 方法此时`try catch`了`dosomething`方法的异常，那这个事务是不能正常回滚的。



### 场景五 同一个类中方法调用

同一个类中方法调用，导致@Transactional失效

开发中避免不了会对同一个类里面的方法调用，比如有一个业务类`AccountService`，它的一个方法`methodA`，`methodA`再调用本类的方法`methodB`（不论方法`methodB`是用public还是private修饰），但方法`methodA`没有声明注解事务，而`methodB`方法有。

那么在外部调用方法`methodA`之后，方法`methodA`的事务是不会起作用的。

会出现这种情况，是使用Spring AOP代理造成的，因为**只有当事务方法被当前类以外的代码调用时，才会由Spring生成的代理对象来管理**。

### 场景六 数据库不支持


事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的`InnoDb`引擎。一旦数据库引擎切换成不支持事务的`MyIsam`，那事务就从根本上失效了。
一般这种情况不会出现。


### 小思考

如果事务在执行，但是突然断网了，客户端断开连接，会发生什么？
