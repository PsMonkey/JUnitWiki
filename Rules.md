> 對應 commit：c96f5bba2b12d3658da987528cb4a32fb8c69e09


Rule
----

rule 允許很靈活地增加或是重新定義每個 test class 中的 test method 的行為。
測試人員可以重用、延伸下列提供的 rule，或是撰寫自己的版本。


### 範例 ###

下面這個 test 使用了 `TemporaryFolder` 與 `ExcepteException` 兩個 rule 以示範其用法：

```java
public class DigitalAssetManagerTest {
	@Rule
	public TemporaryFolder tempFolder = new TemporaryFolder();

	@Rule
	public ExpectedException exception = ExpectedException.none();

	@Test
	public void countsAssets() throws IOException {
		File icon = tempFolder.newFile("icon.png");
		File assets = tempFolder.newFolder("assets");
		createAssets(assets, 3);

		DigitalAssetManager dam = new DigitalAssetManager(icon, assets);
		assertEquals(3, dam.getAssetCount());
	}

	private void createAssets(File assets, int numberOfAssets) throws IOException {
		for (int index = 0; index < numberOfAssets; index++) {
			File asset = new File(assets, String.format("asset-%d.mpg", index));
			Assert.assertTrue("Asset couldn't be created.", asset.createNewFile());
		}
	}

	@Test
	public void throwsIllegalArgumentExceptionIfIconIsNull() {
		exception.expect(IllegalArgumentException.class);
		exception.expectMessage("Icon is null, not a file, or doesn't exist.");
		new DigitalAssetManager(null, null);
	}
}
```


內建的基礎 rule
===============

TemporaryFolder Rule
--------------------

`TemporaryFolder` 可以建立檔案與資料夾，並且在測試結束後刪除掉（無論測試成功或失敗）。
預設狀況下即使 resource 無法被刪除也不會炸出 exception。

```java
public static class HasTempFolder {
	@Rule
	public TemporaryFolder folder = new TemporaryFolder();

	@Test
	public void testUsingTempFolder() throws IOException {
		File createdFile = folder.newFile("myfile.txt");
		File createdFolder = folder.newFolder("subfolder");
		// ...
	}
} 
```


`TemporaryFolder#newFolder(String... folderNames)` 會以遞迴的方式建立暫存目錄。

`TemporaryFolder#newFile()` 會建立一個亂數檔名的檔案、`#newFolder()` 會建立一個亂數名稱的資料夾。

從 4.13 版以後，`TemporaryFolder` 可以選擇性地使用 `AssertionError` 所導致的測試失敗來嚴格檢驗 resource 是否不能被刪除。
要啟用這個功能只能透過 `#builder()`。
預設情況下，為了向下相容，這個嚴格檢驗是關掉的。

```java
@Rule 
public TemporaryFolder folder = TemporaryFolder.builder().assureDeletion().build();
```


ExternalResource Rule
---------------------

`ExternalResource` 跟 `TemporaryFolder` 一樣，是一個基礎的 rule class，
用來在 test 之前設定一個外部的 resource（檔案、socket、server、database 連線...），
並保證之後會銷毀掉：

```java
public static class UsesExternalResource {
	Server myServer = new Server();
  
	@Rule
	public ExternalResource resource = new ExternalResource() {
		@Override
		protected void before() throws Throwable {
			myServer.connect();
		};
    
		@Override
		protected void after() {
			myServer.disconnect();
		};
	};
  
	@Test
	public void testFoo() {
		new Client().run(myServer);
	}
}
```


ErrorCollector Rule
-------------------

`ErrorCollector` 允許在發現第一個問題之後繼續執行。
例如收集*所有* table 中不正確的 row，然後一次回報：

```java
public static class UsesErrorCollectorTwice {
	@Rule
	public ErrorCollector collector= new ErrorCollector();
  
	@Test
	public void example() {
		collector.addError(new Throwable("first thing went wrong"));
		collector.addError(new Throwable("second thing went wrong"));
	}
}
```


Verifier Rule
-------------

`Verifier` 跟 `ErrorCollector` 一樣是一個基礎 class，
如果驗證失敗的話，可以把其他傳入的 test method 轉換成失敗的 test。

```java
private static String sequence;

public static class UsesVerifier {
	@Rule
	public Verifier collector = new Verifier() {
		@Override
			protected void verify() {
			sequence += "verify ";
		}
	};

	@Test
	public void example() {
		sequence += "test ";
	}
  
	@Test
	public void verifierRunsAfterTest() {
		sequence = "";
		assertThat(testResult(UsesVerifier.class), isSuccessful());
		assertEquals("test verify ", sequence);
	}
}
```


TestWatchman/TestWatcher Rule
-----------------------------

`TestWatcher` 從 4.9 版之後取代 `TestWatchman`。
它 implement `TestRule` 而非 `MethodRule`。
（http://junit.org/javadoc/latest/org/junit/rules/TestWatcher.html）

JUnit 在 4.7 版導入 `TestWatchman`，它使用一個現在已經棄置的 `MethodRule`。
（http://junit.org/javadoc/latest/org/junit/rules/TestWatchman.html）

`TestWatcher`（以及棄置的 `TestWatchman`）是 rule 的基礎 class，關注測試行為卻不改動它。
舉例來說，這個 class 會保存所有傳入以及失敗的 test：
     
```java
import static org.junit.Assert.fail; 
import org.junit.AssumptionViolatedException; 
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.TestRule;
import org.junit.rules.TestWatcher;
import org.junit.runner.Description;
import org.junit.runners.model.Statement;

public class WatchmanTest {
	private static String watchedLog;

	@Rule
	public TestRule watchman = new TestWatcher() {
		@Override
		public Statement apply(Statement base, Description description) {
			return super.apply(base, description);
		}

		@Override
		protected void succeeded(Description description) {
			watchedLog += description.getDisplayName() + " " + "success!\n";
		}

		@Override
		protected void failed(Throwable e, Description description) {
			watchedLog += description.getDisplayName() + " " + e.getClass().getSimpleName() + "\n";
		}

		@Override
		protected void skipped(AssumptionViolatedException e, Description description) {
			watchedLog += description.getDisplayName() + " " + e.getClass().getSimpleName() + "\n";
		}

		@Override
		protected void starting(Description description) {
			super.starting(description);
		}

		@Override
		protected void finished(Description description) {
			super.finished(description);
		}
	};

	@Test
	public void fails() {
		fail();
	}

	@Test
	public void succeeds() {
	}
}
```


TestName Rule
-------------

`TestName` rule 可以讓 test method 中取得自己的名字：

```java
public class NameRuleTest {
	@Rule
	public TestName name = new TestName();

	@Test
	public void testA() {
	assertEquals("testA", name.getMethodName());
	}

	@Test
	public void testB() {
		assertEquals("testB", name.getMethodName());
	}
}
```


Timeout Rule
------------

`Timeout` rule 會讓所有 class 中的 test method 有同樣的 timeout 時間：

```java
public static class HasGlobalTimeout {
	public static String log;

	@Rule
	public TestRule globalTimeout = new Timeout(20);

	@Test
	public void testInfiniteLoop1() {
		log+= "ran1";
		for(;;) {}
	}
  
	@Test
	public void testInfiniteLoop2() {
		log+= "ran2";
		for(;;) {}
	}
}
```


ExpectedException Rule
----------------------

`ExpectedException` rule 可以在測試中指定預期的 exception 跟訊息：
    
```java
public static class HasExpectedException {
	@Rule
	public ExpectedException thrown= ExpectedException.none();

	@Test
	public void throwsNothing() {
	}

	@Test
	public void throwsNullPointerException() {
		thrown.expect(NullPointerException.class);
		throw new NullPointerException();
	}

	@Test
	public void throwsNullPointerExceptionWithMessage() {
		thrown.expect(NullPointerException.class);
		thrown.expectMessage("happened?");
		thrown.expectMessage(startsWith("What"));
		throw new NullPointerException("What happened?");
	}
}
```


ClassRule
---------

`ClassRule` 這個 annotation 拓展了 method level rule 的想法，
增加可以影響整個 class 運作的 static field。
所有 `ParentRunner` 的 subclass，包含標準的 `BlockJUnit4ClassRunner` 與 `Suite` 都可以支援 `ClassRule`。

下面這個例子是一個 test suite，在每個 test class 開始前連接 server 然後完成之後斷線。

```java
@RunWith(Suite.class)
@SuiteClasses({A.class, B.class, C.class})
public class UsesExternalResource {
	public static Server myServer= new Server();

	@ClassRule
	public static ExternalResource resource= new ExternalResource() {
		@Override
		protected void before() throws Throwable {
			myServer.connect();
		};

		@Override
		protected void after() {
			myServer.disconnect();
		};
	};
}
```


RuleChain
---------

`RuleChain` 可以指定 `TestRule` 的順序：

```java
public static class UseRuleChain {
	@Rule
	public TestRule chain= RuleChain
		.outerRule(new LoggingRule("outer rule"))
		.around(new LoggingRule("middle rule"))
		.around(new LoggingRule("inner rule"));

	@Test
	public void example() {
		assertTrue(true);
	}
}
```

log 會是


	starting outer rule
	starting middle rule
	starting inner rule
	finished inner rule
	finished middle rule
	finished outer rule


Custom Rule
===========

大多數自訂的 rule 可以用 extend `ExternalResource` rule 的方式來實作。
不過，如果你需要知道 test class 或 method 的資訊，你需要 implement `TestRule`。

```java
import org.junit.rules.TestRule;
import org.junit.runner.Description;
import org.junit.runners.model.Statement;

public class IdentityRule implements TestRule {
	@Override
	public Statement apply(final Statement base, final Description description) {
		return base;
	}
}
```


當然，implement `TestRule` 的威力來自於自訂 constructor、添加 test 中會用到的 method、
把提供的 `Statement` 包進新的 `Statement`。
例如下面這段 test rule 對每個 test 提供了有命名的 logger：

```java
package org.example.junit;

import java.util.logging.Logger;

import org.junit.rules.TestRule;
import org.junit.runner.Description;
import org.junit.runners.model.Statement;

public class TestLogger implements TestRule {
	private Logger logger;

	public Logger getLogger() {
		return this.logger;
	}

	@Override
	public Statement apply(final Statement base, final Description description) {
		return new Statement() {
			@Override
			public void evaluate() throws Throwable {
				logger = Logger.getLogger(description.getTestClass().getName() + '.' + description.getDisplayName());
				
				try {
					base.evaluate();
				} finally {
					logger = null;
				}
			}
		};
	}
}
```


然後可以像這樣子使用：

```java
import java.util.logging.Logger;

import org.example.junit.TestLogger;
import org.junit.Rule;
import org.junit.Test;

public class MyLoggerTest {

	@Rule
	public TestLogger logger = new TestLogger();

	@Test
	public void checkOutMyLogger() {
		final Logger log = logger.getLogger();
		log.warn("Your test is showing!");
	}
}
```