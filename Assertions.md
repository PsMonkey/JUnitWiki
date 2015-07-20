> 對應 commit：9a34a5f1647e8f76b33c3314f9147d032174108

JUnit 對於所有的 primitive type、object 跟（primitive type 或 object 的）array 提供 overload 的 assertion method。
參數的順序是期望值後頭接著實際值。
第一個參數可以是一個在失敗時輸出的字串。
有一個些微不同的 assertion 是 `asserThat`，他的參數有一個選擇性的失敗訊息、實際值、以及 `Matcher` object。
要注意的是，期望值與實際值的順序跟其他 assert method 是顛倒過來的。


範例
----

下列是每一個 assert method 的典型寫法：


```java
import static org.hamcrest.CoreMatchers.allOf;
import static org.hamcrest.CoreMatchers.anyOf;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.not;
import static org.hamcrest.CoreMatchers.sameInstance;
import static org.hamcrest.CoreMatchers.startsWith;
import static org.junit.Assert.assertThat;
import static org.junit.matchers.JUnitMatchers.both;
import static org.junit.matchers.JUnitMatchers.containsString;
import static org.junit.matchers.JUnitMatchers.everyItem;
import static org.junit.matchers.JUnitMatchers.hasItems;

import java.util.Arrays;

import org.hamcrest.core.CombinableMatcher;
import org.junit.Test;

public class AssertTests {
	@Test
	public void testAssertArrayEquals() {
		byte[] expected = "trial".getBytes();
		byte[] actual = "trial".getBytes();
		org.junit.Assert.assertArrayEquals(
			"failure - byte arrays not same", 
			expected, 
			actual
		);
	}

	@Test
	public void testAssertEquals() {
		org.junit.Assert.assertEquals(
			"failure - strings are not equal", 
			"text", 
			"text"
		);
	}

	@Test
	public void testAssertFalse() {
		org.junit.Assert.assertFalse(
			"failure - should be false", 
			false
		);
	}

	@Test
	public void testAssertNotNull() {
		org.junit.Assert.assertNotNull(
			"should not be null", 
			new Object()
		);
	}

	@Test
	public void testAssertNotSame() {
		org.junit.Assert.assertNotSame(
			"should not be same Object", 
			new Object(), 
			new Object()
		);
	}

	@Test
	public void testAssertNull() {
		org.junit.Assert.assertNull(
			"should be null", 
			null
		);
	}

	@Test
	public void testAssertSame() {
		Integer aNumber = Integer.valueOf(768);
		org.junit.Assert.assertSame(
			"should be same", 
			aNumber, 
			aNumber
		);
	}

	// JUnit Matchers assertThat
	@Test
	public void testAssertThatBothContainsString() {
		org.junit.Assert.assertThat(
			"albumen", 
			both(containsString("a")).and(containsString("b"))
		);
	}

	@Test
	public void testAssertThathasItemsContainsString() {
		org.junit.Assert.assertThat(
			Arrays.asList("one", "two", "three"), 
			hasItems("one", "three")
		);
	}

	@Test
	public void testAssertThatEveryItemContainsString() {
		org.junit.Assert.assertThat(
			Arrays.asList(new String[] { "fun", "ban", "net" }), 
			everyItem(containsString("n"))
		);
	}

	// Core Hamcrest Matchers with assertThat
	@Test
	public void testAssertThatHamcrestCoreMatchers() {
		assertThat(
			"good", 
			allOf(equalTo("good"), startsWith("good"))
		);
		
		assertThat(
			"good", 
			not(allOf(equalTo("bad"), equalTo("good")))
		);
		
		assertThat(
			"good", 
			anyOf(equalTo("bad"), equalTo("good"))
		);
		
		assertThat(
			7, 
			not(CombinableMatcher.<Integer> either(equalTo(3)).or(equalTo(4)))
		);
		
		assertThat(
			new Object(), 
			not(sameInstance(new Object()))
		);
	}

	@Test
	public void testAssertTrue() {
		org.junit.Assert.assertTrue(
			"failure - should be true", 
			true
		);
	}
}
```