> 對應 commit：28526eb705534566f21f3cd88e8eef9bb77911ea


test fixture 是一組 object 的固定狀態，用來作為執行測試的基礎。
test fixture 的目的是確保測試的環境是已知且固定的，這樣測試結果才會是可重複的。
例如：

* 準備輸入資料、建立或設定 fake / mock object。
* 載入一組指定、已知的 database 資料
* 複製一些指定的檔案

建立 test fixture 會建立一組初始化為某個狀態的 object。

JUnit 提供 annotation 給 test class 指定在每個測試前後的 fixture、
或是在執行所有 test method 前先執行一次性的 fixture。

總共有四個 fixture annotation：兩個是 class level 的 fixture、兩個是 method level。
class level 是 `@BeforeClass` 跟 `@AfterClass`，method（test）lavel 的是 `@Before` 跟 `@After`。

更深入解釋 fixture、以及如何用 rule 來實作，
可以參見[這篇文章](https://garygregory.wordpress.com/2011/09/25/understaning-junit-method-order-execution/)。

一個實際使用的例子：

```java
package test;

import java.io.Closeable;
import java.io.IOException;

import org.junit.After;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Test;

public class TestFixturesExample {
	static class ExpensiveManagedResource implements Closeable {
		@Override
		public void close() throws IOException {}
	}

	static class ManagedResource implements Closeable {
		@Override
		public void close() throws IOException {}
	}

	@BeforeClass
	public static void setUpClass() {
		System.out.println("@BeforeClass setUpClass");
		myExpensiveManagedResource = new ExpensiveManagedResource();
	}

	@AfterClass
	public static void tearDownClass() throws IOException {
		System.out.println("@AfterClass tearDownClass");
		myExpensiveManagedResource.close();
		myExpensiveManagedResource = null;
	}

	private ManagedResource myManagedResource;
	private static ExpensiveManagedResource myExpensiveManagedResource;

	private void println(String string) {
		System.out.println(string);
	}

	@Before
	public void setUp() {
		this.println("@Before setUp");
		this.myManagedResource = new ManagedResource();
	}

	@After
	public void tearDown() throws IOException {
		this.println("@After tearDown");
		this.myManagedResource.close();
		this.myManagedResource = null;
	}

	@Test
	public void test1() {
		this.println("@Test test1()");
	}

	@Test
	public void test2() {
		this.println("@Test test2()");
	}
}
```


輸出結果會像下面這樣：

	@BeforeClass setUpClass
	@Before setUp
	@Test test2()
	@After tearDown
	@Before setUp
	@Test test1()
	@After tearDown
	@AfterClass tearDownClass
