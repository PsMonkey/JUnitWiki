> 對應 commit：9f24a2b446f6225359739f62b8d7150e94af11ac


略過一個 test
=============

在某些狀況下，你不希望一個 test 失敗，你只是希望略過、關掉這個 test。

在 JUnit 中要略過一個 test，你可以把 method 給註解掉、刪除 `@Test` annotation，
不過 test runner 也就不會回報這個 test。
另外一個方法是加 `@Ignore` 這個 annotation，在 `@Test` 前後都可以。
test runner 會回報略過的 test、執行過的 test、以及失敗的 test。

如果你想要紀錄這個 test 為什麼要略過，`@Ignore` 可以傳入一個字串參數來作到這件事情

```java
@Ignore("Test is ignored as a demonstration")
@Test
public void testSame() {
	assertThat(1, is(1));
}
```