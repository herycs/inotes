# 时间

## 时间处理

### 时区

冬令时和夏令时

- 夏令时、冬令时的出现，是为了充分利用夏天的日照，所以时钟要往前拨快一小时，冬天再把表往回拨一小时。其中夏令时从3月第二个周日持续到11月第一个周日。

    - 冬令时： 北京和洛杉矶时差：16 北京和纽约时差：13

    - 夏令时： 北京和洛杉矶时差：15 北京和纽约时差：12

### 时间戳

- 时间戳（timestamp），一个能表示一份数据在某个特定时间之前已经存在的、 完整的、 可验证的数据,通常是一个字符序列，唯一地标识某一刻的时间。
- 时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。通俗的讲， 时间戳是一份能够表示一份数据在一个特定时间点已经存在的完整的可验证的数据。

### Java中时间API(Java 8)

格林威治时间

- 自1924年2月5日开始，格林尼治天文台负责每隔一小时向全世界发放调时信息。
- 地球每天的自转是有些不规则的，而且正在缓慢减速，因此格林尼治平时基于天文观测本身的缺陷，已经被原子钟报时的协调世界时（UTC）所取代。
- GMT+8表示中国的时间，是因为中国位于东八区，时间上比格林威治时间快8个小时。

### CET、UTC、GMT、CST几种常见时间的含义和关系

- CET
    - 欧洲中部时间（英語：Central European Time，CET）是比世界标准时间（UTC）早一个小时的时区名称之一。它被大部分欧洲国家和部分北非国家采用。冬季时间为UTC+1，夏季欧洲夏令时为UTC+2。
- UTC
    - 协调世界时，又称世界标准时间或世界协调时间，简称UTC，从英文“Coordinated Universal Time”／法文“Temps Universel Cordonné”而来。台湾采用CNS 7648的《资料元及交换格式–资讯交换–日期及时间的表示法》（与ISO 8601类似）称之为世界统一时间。中国大陆采用ISO 8601-1988的国标《数据元和交换格式信息交换日期和时间表示法》（GB/T 7408）中称之为国际协调时间。协调世界时是以原子时秒长为基础，在时刻上尽量接近于世界时的一种时间计量系统。
- GMT
    - 格林尼治标准时间（旧译格林尼治平均时间或格林威治标准时间；英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。
- CST
    - 北京时间，China Standard Time，又名中国标准时间，是中国的标准时间。在时区划分上，属东八区，比协调世界时早8小时，记为UTC+8，与中华民国国家标准时间（旧称“中原标准时间”）、香港时间和澳门时间和相同。當格林威治時間為凌晨0:00時，中國標準時間剛好為上午8:00。
- 关系
    - CET=UTC/GMT + 1小时 
    - CST=UTC/GMT +8 小时 
    - CST=CET+9

### SimpleDateFormat的线程安全性问题

- SimpleDateFormat是Java提供的一个格式化和解析日期的工具类。它允许进行格式化（日期 -> 文本）、解析（文本 -> 日期）和规范化。

- SimpleDateFormat作为一个**非线程安全**的类，被当做了共享变量在多个线程中进行使用，这就出现了线程安全问题。

- 源码

    - format方法在执行过程中，会使用一个成员变量calendar来保存时间，parse方法不是线程安全的。
    - 故，当定义SimpleDateFormat为一个静态变量时多线程情况下会出现问题

- 解决方法

    > 以下方法本质保证线程操作的SimpleDateFormat不受其他线程影响

    - 使用局部变量：定义SimpleDateFormat为局部变量
    - 加同步锁
    - 使用 ThreadLocal
    - JDK 1.8使用使用DateTimeFormatter

### Java 8中的时间处理

> java.time.*包中
>
> `Instant`： 时间戳
>
> `Duration`： 持续时间， 时间差
>
> `LocalDate`： 只包含⽇期
>
> `LocalTime`： 只包含时间
>
> `LocalDateTime`： 包含⽇期和时间
>
> `Period`： 时间段
>
> `ZoneOffset`： 时区偏移量， ⽐如： +8:00
>
> `ZonedDateTime`： 带时区的时间
>
> `Clock`： 时钟， ⽐如获取⽬前美国纽约的时间

- `LocalDate`表⽰⽇期， 年⽉⽇， `LocalTime`表⽰时间， 时分 秒

- 获取当前时间

    - ```java
        LocalDate today = LocalDate.now();
        ```

- 创建指定时间

    - ```java
        LocalDate date = LocalDate.of(2018, 01, 01);
        ```

- 日期间天数差

    - ```java
        Period period = Period.between(LocalDate.of(2018, 1, 5),LocalDate.of(2018, 2, 5));
        ```

        

### 如何在东八区的计算机上获取美国时间

> Java8 中加入了对时区的支持，带时区的时间为分别为：`ZonedDate`、`ZonedTime`、`ZonedDateTime`。
>
> 其中每个时区都对应着 ID，地区ID都为 “{区域}/{城市}”的格式，如`Asia/Shanghai`、`America/Los_Angeles`等。

```java
LocalDateTime now = LocalDateTime.now(ZoneId.of("America/Los_Angeles"));
System.out.println(now);
```

代码无法获取美国时间

```java
System.out.println(Calendar.getInstance(TimeZone.getTimeZone("America/Los_Angeles")).getTime());
```

原因：System.out.println来输出一个时间的时候，他会调用Date类的toString方法，而该方法会读取操作系统的默认时区来进行时间的转换。

故处理方法是修改本地时区为美国

### yyyy和YYYY有什么区别？

> y表示Year ,而Y表示Week Year

不同的国家对于一周的开始和结束的定义是不同的。

- 中国，我们把星期一作为一周的第一天
- 美国，他们把星期日作为一周的第一天

统一时间

- 国际标准化组织的国际标准ISO 8601是日期和时间的表示方法，全称为《数据存储和交换形式·信息交换·日期和时间的表示方法》。

    > 在 ISO 8601中。对于一年的第一个日历星期有以下四种等效说法：
    >
    > - 1，本年度第一个星期四所在的星期；
    > - 2，1月4日所在的星期；
    > - 3，本年度第一个至少有4天在同一星期内的星期；
    > - 4，星期一在去年12月29日至今年1月4日以内的星期；

结论

- 我们要表示日期的时候，一定要使用 yyyy-MM-dd 而不是 YYYY-MM-dd ，这两者的返回结果大多数情况下都一样，但是极端情况就会有问题了。