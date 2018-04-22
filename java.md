1. 将常量放在接口中，通过继承该接口，调用常量

   ```java
   public interface ClassConstants{
      int CONSUMER = 1;//接口中变量默认为 static final
      int DISPLAY = 2;
   }
   ```

   ​

2. 该日志类可打印记录日志的类，方法，及调用行数

   ```java
   public class Log {

       public static void log(Object o) {
           System.out.println(Common.SIMPLE_DATE_FORMAT.format(new Date()) + "\t" + Thread.currentThread().getStackTrace()[2] + ":" + o);
       }
   }
   20170726 12:07:22 127	com.li.dao.Test.main(Test.java:17):main
   20170726 12:07:22 128	com.li.dao.Test.test(Test.java:23):test
   20170726 12:07:22 128	com.li.dao.Test.subTest(Test.java:28):subTest
   20170726 12:07:22 128	com.li.util.Common.test(Common.java:12):common
   ```

3. 生成javadoc 中文乱码javadoc 工具加上参数

  ```
  -encoding UTF-8 -charset UTF-8
  ```

4. 查看Class是否是基本类型

   ```java
   clasz.isPrimitive();
   ```

   ​

5. 日志规范

   1. **修改（包括新增）操作必须打印日志**
   2. **条件分支必须打印条件值，重要参数必须打印**
   3. **数据量大的时候需要打印数据量**
   4. **代码开发测试完成之后不要急着提交，先跑一遍看看日志是否看得懂**

   ​

6. 批量反编译

   ```shell
   ls *.jar|xargs -n1 jar xvf

   jad -b  -ff -o -r -sjava -dsrc "lib/**/*.class"
   ```



   ​

7.  void类型的范型方法

```java
private void test() {
	this.<String>set("1");
}

private <T> void set(T t) {
    System.out.println(t);
}
```

8. string替换 正则组

```java
String s = "HelloWorldMyNameIsCarl".replaceAll("(.)([A-Z])", "$1_$2");
```

9. 断言**ExpectedException**

```java
public class Student {
  public boolean canVote(int age) {
      if (i<=0) throw new IllegalArgumentException("age should be +ve");
      if (i<18) return false;
      else return true;
  }
}

public class TestStudent{

	@Rule
	public ExpectedException thrown= ExpectedException.none();

	@Test
	public void canVote_throws_IllegalArgumentException_for_zero_age() {
		Student student = new Student();
		thrown.expect(IllegalArgumentException.class);
		thrown.expectMessage("age should be +ve");
		student.canVote(0);
	}
}

```

