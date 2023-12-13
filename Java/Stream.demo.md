---
layout: default
title: Java Stream API
parent: Java
nav_order: 90
---


## Stream中常用方法

Stream流的api方法
 
- stream(), parallelStream()
- filter(过滤)
- sorted(排序), distinct(去重) 
- findAny(返回流中的任意一个), findFirst(返回流中的第一个),limit(截取), skip(跳过)
- anyMatch(是否存在满足指定条件), allMatch(所有元素是否都满足指定条件)
- groupingBy
- count, min, max, summaryStatistics，map(计算或转换), reduce(归约)
- flatMap() - 将多个Stream连接成一个Stream
- collect(Collectors.toList())收集



>reduce：归约，也称缩减，顾名思义，是把一个流缩减成一个值，能实现对集合求和、求乘积和求最值操作。
>
>惰性求值（如果没有终结操作，没有中间操作是不会得到执行的）
>
>流是一次性的（一旦一个流对象经过一个终结操作后。这个流就不能再被使用）
>
>不会影响原数据（我们在流中可以多数据做很多处理。但是正常情况下是不会影响原来集合中的元素的。这往往也是我们期望的）


## DEMO

准备数据
 
```java
public class UserStreamDemo {

    public List<UserVo> initUserList() {
        List<UserVo> userVoList = Collections.EMPTY_LIST;
        UserVo userVo = null;
        for (int x = 1; x <= 100; x++) {
            userVo = new UserVo();
            userVo.setId(x);
            userVo.setName(PersonUtils.randomChineseName());
            userVo.setAge(PersonUtils.randomYoungAge());
            userVo.setAmount(new BigDecimal(PersonUtils.randomAmount4Bit()));
            if (x <= 30) {
                userVo.setVipLeveL("A");
            } else if (x <= 60) {
                userVo.setVipLeveL("B");
            } else {
                userVo.setVipLeveL("C");
            }
            userVoList.add(userVo);
        }
        return userVoList;
    }
}
```

### 1、filter 过滤

```java

public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();
        // 过滤 年龄   filter
        List<UserVo> collect = userVoList.stream().filter(n -> (n.getAge() < 30)).collect(Collectors.toList());
        collect.forEach(System.out::println);
        // 过滤   filter
        List<UserVo> collect2 = userVoList.stream().filter(u -> u.getName().startsWith("张")).collect(Collectors.toList());
        collect2.forEach(System.out::println);
        List<UserVo> last = userVoList.stream().filter(n -> (n.getAge() < 30)).filter(u -> u.getName().startsWith("张")).collect(Collectors.toList());

        //把 -年龄大于20人--里面名字提取出来放入新的集合。
        List<String> list20 = userVoList.stream().filter(item -> item.getAge() > 20).map(item -> item.getName()).collect(Collectors.toList());
        list20.forEach(System.out::println);

        //获取 所有用户名称列表
        List<String> nameList = userVoList.stream().map(UserVo::getName).collect(Collectors.toList());

    }
}
```

### 2、find、limit、skip

```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();

        //findFirst(): 返回流中的第一个元素，如果流为空则返回一个空的Optional对象。
        Optional<UserVo> first = userVoList.stream().filter(item -> item.getAge() >= 20 && item.getAge() <= 24).findFirst();
        System.out.println(first.get());

        //findAny(): 返回流中的任意一个元素，如果流为空则返回一个空的Optional对象
        Optional<UserVo> any = userVoList.stream().filter(item -> item.getAge() > 20).findAny();
        System.out.println(any.get());
        //获取用户列表，要求跳过第1条数据后，取前3条数据
        List<UserVo> userList = userVoList.stream()
                .skip(1)
                .limit(3)
                .collect(Collectors.toList());
        userList.forEach(System.out::println);

        // 获取 所有用户名称列表 
        List<String> distinctList = userVoList.stream().map(UserVo::getName).collect(Collectors.toList());
    }
}
```

### 3、sorted、distinct

```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();

        //sorted排序 按照年龄从小到大排序 （升序）
        List<UserVo> sortedList1 = userVoList.stream().sorted((o1, o2) -> o1.getAge() - o2.getAge()).collect(Collectors.toList());
        List<UserVo> sortedList2 = userVoList.stream().sorted(Comparator.comparingInt(UserVo::getAge)).collect(Collectors.toList());
        sortedList2.forEach(System.out::println);
        //降序： reversed
        List<UserVo> sortedList3 = userVoList.stream().sorted(Comparator.comparingInt(UserVo::getAge).reversed()).collect(Collectors.toList());
        

        // 获取 所有用户名称列表 并 distinct 去重
        List<String> distinctList = userVoList.stream().map(UserVo::getName).distinct().collect(Collectors.toList());
    }

}
```

### 4、Match 匹配 

```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();

        //anyMatch 判断流中是否存在满足指定条件的元素，一旦找到满足条件的元素，立即返回true，不再继续处理剩余元素。
        //判断有没有姓名为张小菜的
        boolean b = userVoList.stream().anyMatch(a -> a.getName().equals("张小菜"));
        System.out.println(b);

        //allMatch 判断流中的所有元素是否都满足指定条件，一旦发现有不满足条件的元素，立即返回false，不再继续处理剩余元素。
        //判断是不是所有用户的级别都是为 A
        boolean b2 = userVoList.stream().allMatch(a -> a.getVipLeveL().equals("A"));
        System.out.println(b2);
        //noneMatch 判断中的所有元素的用户名称是否存在不包含“张小菜”字段
        boolean b3 = userVoList.stream().noneMatch(user -> user.getName().contains("张小菜"));
        System.out.println(b3);        
    }
}
```

### 5、groupingBy 分组 

```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();

        // groupingBy 按 vipLevel 分组
        Map<String, List<UserVo>> listMap = userVoList.stream().collect(Collectors.groupingBy(UserVo::getVipLeveL));

        // groupingBy 按 vipLevel 并行分组
        Map<String, List<UserVo>> listMap2 = userVoList.parallelStream().collect(Collectors.groupingByConcurrent(UserVo::getVipLeveL));

        //多级 groupingBy  根据vipLevel和性别对用户列表进行分组
        Map<String,Map<String,List<UserVo>>> userMap = userVoList.stream()
                .collect(Collectors.groupingBy(UserVo::getVipLeveL,Collectors.groupingBy(UserVo::getGender)));
        //遍历分组后的结果
        userMap.forEach((key1, map) -> {
            System.out.println(key1 + "：");
            map.forEach((key2,user)->
            {
                System.out.println(key2 + "：");
                user.forEach(System.out::println);
            });
            System.out.println("-------------------------------------------------------");
        });
        //根据部门进行分组，汇总各个部门用户的平均年龄
        Map<String, Double> groupingByMap = userVoList.stream().collect(Collectors.groupingBy(UserVo::getVipLeveL, Collectors.averagingInt(UserVo::getAge)));
        //遍历分组后的结果
        groupingByMap.forEach((key, value) -> {
            System.out.println(key + "的平均年龄：" + value);
        });


    }
}
```


### 6、count, min, max, summaryStatistics，map, mapToInt、reduce

>count, counting, min, max, sum, average, summaryStatistics，map, mapToInt、reduce

```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();
        // count 推荐
        long count = userVoList.stream().filter(item -> item.getAge() >= 20 && item.getAge() <= 24).count();
        System.out.println(count);
        //统计 vipLevel = C 的人数，使用 counting()方法进行统计
        Long CCount = userVoList.stream().filter(user -> user.getVipLeveL() == "C").collect(Collectors.counting());
        //统计金额大于5000元的人数
        Long amountCount = userVoList.stream().filter(user -> user.getAmount().compareTo(BigDecimal.valueOf(5000)) == 1).count();
        // summingInt 计算年龄总和
        int sumAge = userVoList.stream().collect(Collectors.summingInt(UserVo::getAge));
        // sum 推荐
        int sumAge2 = userVoList.stream().mapToInt(UserVo::getAge).sum();

        //找出年龄最大和最小
        // max min
        UserVo u1 = userVoList.stream().max((Comparator.comparingInt(UserVo::getAge))).get();
        UserVo u2 = userVoList.stream().min(((o1, o2) -> o1.getAge() - o2.getAge())).get();
        // reduce 取 年龄最大和最小
        int maxVal = userVoList.stream().map(UserVo::getAge).reduce(Integer::max).get();
        int minVal = userVoList.stream().map(UserVo::getAge).reduce(Integer::min).get();
        // mapToInt max 取 年龄最大和最小
        int maxVal2 = userVoList.stream().mapToInt(UserVo::getAge).max().getAsInt();
        int minVal2 = userVoList.stream().mapToInt(UserVo::getAge).min().getAsInt();
        System.out.println(" max = " + u1.getAge() + ", maxVal = " + maxVal + ", maxVal2 = " + maxVal2);
        System.out.println(" min = " + u2.getAge() + ", minVal = " + minVal + ", minVal2 = " + minVal2);
        // average 用户列表中年龄的 平均值
        double aveAge = userVoList.stream().collect(Collectors.averagingDouble(UserVo::getAge));
        double aveVal = userVoList.stream().mapToInt(UserVo::getAge).average().getAsDouble();
        //打印结果
        System.out.println("年龄总和：" + sumAge);
        System.out.println("平均年龄：" + aveVal);

        //获取IntSummaryStatistics对象
        IntSummaryStatistics ageStatistics = userVoList.stream().collect(Collectors.summarizingInt(UserVo::getAge));
        //统计：最大值、最小值、总和、平均值、总数
        System.out.println("最大年龄：" + ageStatistics.getMax());
        System.out.println("最小年龄：" + ageStatistics.getMin());
        System.out.println("年龄总和：" + ageStatistics.getSum());
        System.out.println("平均年龄：" + ageStatistics.getAverage());
        System.out.println("用户总数：" + ageStatistics.getCount());

        // 汇总所有用户的总金额
        //最高薪资
        BigDecimal maxSalary = userVoList.stream().map(UserVo::getAmount).max((x1, x2) -> x1.compareTo(x2)).get();
        //最低薪资
        BigDecimal minSalary = userVoList.stream().map(UserVo::getAmount).min((x1, x2) -> x1.compareTo(x2)).get();
        //薪资总和
        BigDecimal sumSalary = userVoList.stream().map(UserVo::getAmount).reduce(BigDecimal.ZERO, BigDecimal::add);
        //平均薪资
        BigDecimal avgSalary = userVoList.stream().map(UserVo::getAmount).reduce(BigDecimal.ZERO, BigDecimal::add).divide(BigDecimal.valueOf(userVoList.size()), 2, BigDecimal.ROUND_HALF_UP);
        //打印统计结果
        System.out.println("最高薪资：" + maxSalary + "元");
        System.out.println("最低薪资：" + minSalary + "元");
        System.out.println("薪资总和：" + sumSalary + "元");
        System.out.println("平均薪资：" + avgSalary + "元");

    }
}
```


### 7、flatMap


```java
public class UserStreamDemo {

    private void streamDemo() {
        List<UserVo> userVoList = initUserList();

        // 使用flatMap()将流中的每一个元素连接成为一个流
        //创建城市
        List<String> cityList = new ArrayList<String>();
        cityList.add("北京；上海；深圳；");
        cityList.add("广州；武汉；杭州；");
        //分隔城市列表，使用 flatMap() 将流中的每一个元素连接成为一个流。
        cityList = cityList.stream()
                .map(city -> city.split("；"))
                .flatMap(Arrays::stream)
                .collect(Collectors.toList());
        //遍历城市列表
        cityList.forEach(System.out::println);
   }

}
```

### 8、collect

```java
public class UserStreamDemo {
    public void List2Map() {

        List<UserVo> userList = initUserList();
        // List 转成 Map
        // key 是属性，value 是属性
        Map<Long, String> userMap1 = userList.stream().collect(Collectors.toMap(UserVo::getId, UserVo::getName));
        // key 是属性，value 是对象
        Map<Long, UserVo> userMap2 = userList.stream().collect(Collectors.toMap(UserVo::getId, UserVo -> UserVo));
        Map<Long, UserVo> userMap3 = userList.stream().collect(Collectors.toMap(UserVo::getId, Function.identity()));
        // 更安全的写法
        Map<Long, UserVo> userIdMap2 = Optional.ofNullable(userList).orElse(new ArrayList<UserVo>())
                .stream().collect(Collectors.toMap(item -> item.getId(), item -> item));
        Map<Long, UserVo> userIdMap3 = Optional.ofNullable(userList).orElse(new ArrayList<UserVo>())
                .stream().collect(Collectors.toMap(UserVo::getId, Function.identity()));
        // key 是属性，value 是对象,  出现重复key的处理策略，使用新的覆盖
        // Collectors.toMap方法的第三个参数（key冲突处理函数），那么在key重复的情况下该方法会报出【Duplicate Key】的错误导致Stream流异常终止
        Map<Long, UserVo> userMap4 = userList.stream().collect(Collectors.toMap(UserVo::getId, Function.identity(), (oldValue, newValue) -> newValue));
        // 生成指定类型的 Map
        Map<Long, UserVo> userIdMap = Optional.ofNullable(userList).orElse(new ArrayList<UserVo>())
                .stream().collect(Collectors.toMap(UserVo::getId, Function.identity(), (key1, key2) -> key1, IdentityHashMap::new));

        // List的元素为Map，需要转换的Map的key为元素Map的某个key，需要转换的Map的value为整个元素Map
        List<Map<String, Object>> mapList = new ArrayList<Map<String, Object>>();
        Map<String, Map<String, Object>> objMap = Optional.ofNullable(mapList).orElse(new ArrayList<Map<String, Object>>())
                .stream().collect(Collectors.toMap(map -> String.valueOf(map.get("key")), item -> item));

        List<Map<String, Object>> mapList2 = new ArrayList<Map<String, Object>>();
        //重复key的处理策略
        Map<String, Map<String, Object>> lisMap = Optional.ofNullable(mapList2).orElse(new ArrayList<Map<String, Object>>())
                .stream().collect(Collectors.toMap(map -> String.valueOf(map.get("key")), item -> item, (key1, key2) -> key1));
        // 指定 Map类型
        Map<String,List<Map<String,Object>>> userScheduleListMap2=Optional.ofNullable(mapList2).orElse(new ArrayList<Map<String,Object>>()).stream()
                .collect(Collectors.groupingBy(item-> String.valueOf(item.get("key")), LinkedHashMap::new,Collectors.toList()));

    }
}
```

### 9、相同属性对象合并

```java
public class UserStreamDemo {

    private void streamDemo() {

        //合并同一个集合内相同属性对象
        List<Student> studentList = new ArrayList<>();
        List<Student> result = new ArrayList<>();
        studentList.add(new Student("张三","16",80));
        studentList.add(new Student("张三","16",90));
        studentList.add(new Student("张三","16",90));
        studentList.add(new Student("李四","18",70));

        Map<String, List<Student>> tmpMap = studentList.parallelStream().collect(Collectors.groupingBy(o -> (o.getName() + o.getAge()), Collectors.toList()));
        tmpMap.forEach(
        (id,transfer)->{
                transfer.stream().reduce((x, y) -> new Student(x.getName(), x.getAge(), (x.getScore() + y.getScore()))).ifPresent(result::add);
        });
        System.out.println("student ="+ studentList);
   }
}

@Data
@AllArgsConstructor
class Student{
    private String name ;
    private String age ;
    private int score ;
}
```

### 10、其他
