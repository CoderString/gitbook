### commons-lang3包API总结

在Java项目的开发过程中，针对各种类型的参数在添加进数据库之前，一般需要对这些参数的合法性进行一定约束规则的校验，如果全部自己实现，一则代码冗余，其次实现的代码还可能存在一些逻辑漏洞，针对这部分的工作，我们完全可以借助一些优秀的开源包来帮助我们完成这部分的工作，比如下面介绍的的apache开源的commons-lang3包。

---


### 依赖引入方式

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.9</version>
</dependency>
```

---

### 核心工具类

- StringUtils

| 方法        | 描述    | 
| --------   | :-----| 
| StringUtils.isBlank(CharSequence cs)        | 判断目标字符串为空或者为null，空格也当作为空      |
| StringUtils.isNotBlank(CharSequence cs)  | 判断目标字符串不为空且不为null，空格也当作为空     |
| StringUtils.isEmpty(CharSequence cs)        | 判断目标字符串不为空且不为null，空格也当作为空      |
| StringUtils.isNotEmpty(CharSequence cs)        | 判断目标字符串不为空且不为null，空格不当作空      |
| StringUtils.isNumeric(CharSequence cs)       | 判断字符串是否是数字，不忽略空格      |
| StringUtils.isNumericSpace(CharSequence cs)        | 判断字符串是否是数字，忽略空格      |
| StringUtils.isAlpha(CharSequence cs)        | 判断字符串是否是希腊字母，不忽略空格      |
| StringUtils.isAlphaSpace(CharSequence cs)        | 判断字符串是否是希腊字母，忽略空格      |
| StringUtils.isAlphanumeric(CharSequence cs)        | 判断字符串是否是希腊字母与数字组成，不忽略空格      |
| StringUtils.isAlphanumericSpace(CharSequence cs)       | 判断字符串是否是希腊字母与数字组成，忽略空格      |
| StringUtils.isAllLowerCase(CharSequence cs        | 判断字符串是否全是小写字母      |
| StringUtils.isAllUpperCase(CharSequence cs)       | 判断字符串是否全大写字母      |
| StringUtils.trim(String str)        | 去掉字符串前后空格。为空时返回空，为 null 时返回 null(不是字符串null，防止空指针异常)      |
| StringUtils.trimToNull(String str)        | 去掉字符串前后空格。当为空时，返回 null(不是字符串null，防止空指针异常)      |
| StringUtils.trimToEmpty(String str)        | 去掉字符串前后空格。当为 null 时，返回空      |

---

- ArraysUtils

| 方法        | 描述    | 
| --------   | :-----| 
| ArrayUtils.addAll(array1, array2))        | 合并两个数组array1、array2，输出合并后的数组      |
| ArrayUtils.clone(array1)  | 克隆某个数组     |
| ArrayUtils.contains(array1, "a")        | 数组是否包含某个元素      |
| ArrayUtils.getLength(array2)        | 获取数组的长度      |
| ArrayUtils.indexOf(array2, "e")        | 某个元素在数组所在的位置，有相同的取第一个      |
| ArrayUtils.isEmpty(array1)        | 判断一个数组是否为空      |
| ArrayUtils.isEquals(array1, array2)        | 判断两个数组的是否相等      |
| ArrayUtils.isSameLength(array1, array2)       |  判断两个数组的长度是否相等      |
| ArrayUtils.getLength(array2)        | 获取数组的长度      |
| ArrayUtils.isSameType(array1, array2)        |  两个数组的类型是否相同      |
| ArrayUtils.lastIndexOf(array3, 3)        | 获取某个元素在数组中最后一个位置      |
| ArrayUtils.removeElements(array3, 1)        | 移除数组的某个元素      |
| ArrayUtils.reverse(array3)        | 将某个数组元素倒序      |
| ArrayUtils.subarray(array3, 1, 2)        | 截取某个数组返回新的数组      |
| Integer[] object = ArrayUtils.toObject(array3)     | int[] 数组转换成 Integer[]     |
| int[] primitive = ArrayUtils.toPrimitive(array4)        | Integer[] 数组转换成 int[] |

---

- BooleanUtils

| 方法        | 描述    | 
| --------   | :-----| 
| ArrayUtils.addAll(array1, array2))        | 合并两个数组array1、array2，输出合并后的数组      |
| ArrayUtils.clone(array1)  | 克隆某个数组     |

---

- NumberUtils

| 方法        | 描述    | 
| --------   | :-----| 
| NumberUtils.isNumber(String str)        | 用于判断字符串中是否是数字，返回的结果是true或者false（废弃） |
| NumberUtils.isCreatable(String str) | 用于判断字符串中是否是数字，返回的结果是true或者false     |
| NumberUtils.isDigits(String str) | 用于判断字符串中是否全是数字，不能检验小数的情况，返回的结果是true或者false     |
| NumberUtils.toInt(String str) | 字符串转换为int类型    |
| NumberUtils.toFloat(String str) | 字符串转换为float类型    |
| NumberUtils.toDouble(String str) | 字符串转换为double类型    |
| NumberUtils.toShort(String str) | 字符串转换为short类型    |
| NumberUtils.toByte(String str) | 字符串转换为byte类型    |
| NumberUtils.max(new int[]{3,5,6}) | 找出最大的一个    |
| NumberUtils.min(new int[]{3,5,6}) | 找出最小的一个    |
| NumberUtils.compare(int x, int y) | 比较两个数的大小，相等返回0， 小于返回-1，大于返回1    |

---

- DateUtils

| 方法        | 描述    | 
| --------   | :-----| 
| DateUtils.addYears(Date date, int amount)         | 向新返回日期对象添加若干年，原始日期不变 |
| DateUtils.addMonths(Date date, int amount)         | 向新返回日期对象添加若干月，原始日期不变 |
| DateUtils.addWeeks(Date date, int amount)         | 向新返回日期对象添加若干年，原始日期不变 |
| DateUtils.addDays(Date date, int amount)         | 向新返回日期对象添加若干天，原始日期不变 |
| DateUtils.addHours(Date date, int amount)         | 向新返回日期对象添加若干小时，原始日期不变 |
| DateUtils.addMinutes(Date date, int amount)         | 向新返回日期对象添加若干分钟，原始日期不变 |
| DateUtils.addSeconds(Date date, int amount)         | 向新返回日期对象添加若干秒，原始日期不变 |
| DateUtils.addMinutes(Date date, int amount)         | 向新返回日期对象添加若干分钟，原始日期不变 |
| DateUtils.setYears(Date date, int amount)         | 设置日期中年为指定的值，原始日期不变 |
| DateUtils.setMonths(Date date, int amount)         | 设置日期中月为指定的值，原始日期不变 |
| DateUtils.setDays(Date date, int amount)         | 设置日期中日为指定的值，原始日期不变 |
| DateUtils.setHours(Date date, int amount)      | 设置日期中小时为指定的值，原始日期不变 |
| DateUtils.setMinutes(Date date, int amount)         | 设置日期中分钟为指定的值，原始日期不变 |
| DateUtils.setSeconds(Date date, int amount)         | 设置日期中秒为指定的值，原始日期不变 |
| DateUtils.isSameDay(date, date)         | 比较年月日是否相等 |
| DateUtils.isSameInstant(date, date)         | 比较时间毫秒数是否相等 |
| DateUtils.isSameDay(calendar, calendar)         | 比较日历年月日是否相等 |
| DateUtils.isSameInstant(calendar, calendar)         | 比较日历毫秒数是否相等 |
| DateUtils.isSameLocalTime(calendar, calendar)         |  判断是否是同一个本地时间 |
| DateUtils.toCalendar(date)         |  Date转为Calendar |
| DateUtils.toCalendar(date, TimeZone.getTimeZone("Asia/Shanghai"))        |  Date转为指定时区的Calendar |
| Date DateUtils.parseDate(final String str, final Locale locale, final String... parsePatterns)         |  该方法中，Locale locale 参数指定语言环境，如果为空，则默认为当前系统的语言环境 |

---

- DateFormatUtils

| 方法        | 描述    | 
| --------   | :-----| 
| DateFormatUtils.format(date,"yyyy-MM-dd HH:mm:ss")        | Date类型转化为日期字符串      |

---

- ObjectUtils

| 方法        | 描述    | 
| --------   | :-----| 
| ObjectUtils.allNotNull(Object… values)         | 检查所有元素是否为空,返回一个boolean      |
| ObjectUtils.anyNotNull() | 如果有一个元素不为空返回true     |
| notEqual(Object object1, Object object2)  | 判断两个对象不相等，返回一个boolean     |
| ObjectsUtils.isEmpty(final Object object)  | 检查对象是否为空，返回一个boolean     |
| ObjectUtils.isNotEmpty(final Object object)  | 检查对象是否不为空，返回一个boolean     |

---

### 参考资料

1、https://blog.csdn.net/wangmx1993328/article/details/102488632

2、https://blog.csdn.net/shy415502155/article/details/89350739

3、https://www.cnblogs.com/muxi0407/p/11947562.html

4、https://blog.csdn.net/han12398766/article/details/92237210