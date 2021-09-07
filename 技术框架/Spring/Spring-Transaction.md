

#### 事务的理解

对事务的理解：事务是指修改数据库数据时，需要满足所有的操作要满足一致性，要么全都操作，要么全都不操作。例如，当我买一个商品时，大致的步骤如下：

1.数据库中商品的数量减一

2.我的账户余额减去商品的价格
当执行到第二步时，若发现我的余额不够支付该商品，本次购买便失败，系统应该恢复原来的商品数量（即需要回滚），这便是一个典型的事务，库存减一和账户消费必须一致，要么一起执行成功，要么都不执行。

#### REQUIRED

REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。


#### REQUIRES_NEW

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
```

```java
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

#### SUPPORTS

SUPPORTS:如果有事务在运行，当前的方法就在这个事务内运行，否则它可以不运行在事务中


#### NOT_SUPPORTED

NOT_SUPPORTED ：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。


#### NEVER

NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。


#### MANDATORY

MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。


#### NESTED

NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与REQUIRED类似的操作。拥有多个可以回滚的保存点，内部回滚不会对外部事务产生影响。只对DataSourceTransactionManager有效