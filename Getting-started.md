> 對應 commit：f0f0fc9fffdb5c582a9cc4c557a6c4a3f37bf4ba


這個小型範例展示如何寫一個 unit test。
你需要安裝 JDK 跟一個文字編輯器。
（通常會建議你用 build tool 來建置軟體與執行測試）


前置準備
--------

建立一個新的目錄，假設叫做 `junit-example`，然後從 JUnit 的[版本頁面]下載目前的 `junit-4.XX.jar` 以及 [Hamcrest] 到這個目錄裡頭。
切到 `junit-example` 目錄。
所有檔案都建立在這個目錄下、也在這個目錄裡頭執行指令。

[版本頁面]: https://github.com/junit-team/junit/releases
[Hamcrest]: http://search.maven.org/remotecontent?filepath=org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar


建立要測試的 class
------------------

新建一個 `Calculator.java` 的檔案，內容是下面這段程式碼：

```java
public class Calculator {
	public int evaluate(String expression) {
		int sum = 0;
		
		for (String summand: expression.split("\\+")) {
			sum += Integer.valueOf(summand);
		}
			
		return sum;
	}
}
```


然後 compile 這個 class：

	javac Calculator.java


Java compiler 會產生 `Calculator.class`。


建立 test
---------

新建一個 `CalculatorTest.java` 的檔案，內容是下面這段程式碼：

```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class CalculatorTest {
	@Test
	public void evaluatesExpression() {
		Calculator calculator = new Calculator();
		int sum = calculator.evaluate("1+2+3");
		assertEquals(6, sum);
	}
}
```

compile 這個 test，在 Linux 或 MacOS 上是

    javac -cp .:junit-4.XX.jar CalculatorTest.java

	
在 Windows 上是

    javac -cp .;junit-4.XX.jar CalculatorTest.java


Java compiler 會產生 `CalculatorTest.class`。
 

執行 test
---------

用 command line 執行 test，在 Linux 或 MacOS 上是

    java -cp .:junit-4.XX.jar:hamcrest-core-1.3.jar org.junit.runner.JUnitCore CalculatorTest


在 Windows 上是

    java -cp .;junit-4.XX.jar;hamcrest-core-1.3.jar org.junit.runner.JUnitCore CalculatorTest


輸出結果會是

    JUnit version 4.12
    .
    Time: 0,006
    
    OK (1 test)


那一個 `.` 代表執行了一個 test，最後一行的 `OK` 告訴你 test 執行成功。


讓 test 失敗
------------

為了讓 test 失敗，我們把 `Calculator.java` 的這行：

    sum += Integer.valueOf(summand);

	
改成這行

    sum -= Integer.valueOf(summand);

	
然後重新 compile 這個 class：

    javac Calculator.java

	
再次執行這個 test，在 Linux 跟 MacOS：

    java -cp .:junit-4.XX.jar:hamcrest-core-1.3.jar org.junit.runner.JUnitCore CalculatorTest

	
在 Windows：

    java -cp .;junit-4.XX.jar;hamcrest-core-1.3.jar org.junit.runner.JUnitCore CalculatorTest


現在 test 會失敗，輸出結果會是：

    JUnit version 4.12
    .E
    Time: 0,007
    There was 1 failure:
    1) evaluatesExpression(CalculatorTest)
    java.lang.AssertionError: expected:<6> but was:<-6>
      at org.junit.Assert.fail(Assert.java:88)
      ...
    
    FAILURES!!!
    Tests run: 1,  Failures: 1


JUnit 告訴你哪一個 test 失敗了（`evaluatesExpression(CalculatorTest)`），以及錯誤是什麼：

    java.lang.AssertionError: expected:<6> but was:<-6>
