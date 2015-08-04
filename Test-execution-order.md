> 對應 commit：6f2550fc1f6dafe9407b0ebb8ac3b81ec9f2ccf6


test 執行順序
=============

JUnit 的設計無法指定 test method 的呼叫順序。
到目前為止，是透過 reflection API 回傳的順序依次觸發。
不過，使用 JVM 的順序不太明智，因為 Java 平台沒有指明用什麼特定的順序；
事實上，JDK 7 的回傳順序多少帶有亂數的成份。
當然，寫得很好的 test 碼不會假設有先後順序、但是有些會；
在某些平台上如果測試會失敗，可預期的話會是比較好的。

從 4.11 版以後，JUnit 預設會用決定（但無法預測）一種順序（`MethodSorter.DEFAULT`）。
要改變 test 的執行順序，就在你的 test class 掛上 `@FixMethodOrder` 這個 annotation，
然後指定其中一個可用的 MethuodSorter：

* `@FixMethodOrder(MethodSorters.JVM)`：使用 JVM 回傳的順序，每次執行時的順序可能不盡相同。
* `@FixMethodOrder(MethodSorters.NAME_ASCENDING)`：
	用字典（*譯註：原文是 lexicographic 不是 alphabetical*）順序排序 method 名稱。


範例
----

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class TestMethodOrder {

	@Test
	public void testA() {
		System.out.println("first");
	}
	
	@Test
	public void testB() {
		System.out.println("second");
	}
	
	@Test
	public void testC() {
		System.out.println("third");
	}
}
```


上面這段程式碼會用 method 的名字由小到大依序執行。