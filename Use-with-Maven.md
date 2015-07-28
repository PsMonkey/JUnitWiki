> 對應 commit：18f6cd07c7a47b65b8b40fb6cbcbd88ca6af0090


Maven Dependency
================

```xml
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.12</version>
	<scope>test</scope>
</dependency> 
```


執行
----

參閱 [Maven Surefire plugin](http://maven.apache.org/plugins/maven-surefire-plugin/) 與 
[Maven Failsafe Plugin](http://maven.apache.org/surefire/maven-failsafe-plugin/).


在 Maven 中用 JUnit 與 Hamcrest
-------------------------------

### 4.11 版以後 ###
到目前為止有兩個 JUnit 的 Maven artifact：`junit:junit-dep` 跟 `junit:junit`。
以 Maven 的角度，只有前面那個比較合理，因為他沒有包含 Hamcrest 的 class、但有宣告 Hamcrest Maven artifact 的 dependency。
後面那個包含了 Hamcrest 的 class、卻非常不像 Maven。

從這個版本（4.11）開始，你應該在過去用 `junit:junit-dep` 的時候改用 `junit:junit`。
如果你始終 reference 到 `junit:junit-dep`，Maven 會自動幫你導向到新的 `junit:junit` 並提醒你有一個 warning 要修正。


### 4.10 版以前 ###

在 `pom.xml`，宣告 `junit-dep` 的 dependency，然後複寫 transitive dependency 為 `hamcrest-core`，
這樣你就能夠用所有 Hamcrest 的 matcher 了。

```xml
<dependency>
	<groupId>org.hamcrest</groupId>
	<artifactId>hamcrest-core</artifactId>
	<version>1.3</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit-dep</artifactId>
	<version>4.10</version>
	<scope>test</scope>
</dependency>         
<dependency>
	<groupId>org.hamcrest</groupId>
	<artifactId>hamcrest-library</artifactId>
	<version>1.3</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-core</artifactId>
	<version>1.9.0</version>
	<scope>test</scope>
</dependency>
```


從 Surefire 產生的 XML 找出最慢的 test
--------------------------------------

下面那行程式是出自於[這裡](http://stackoverflow.com/questions/5094410/how-to-list-the-slowest-junit-tests-in-a-multi-module-maven-build)。

報告以 test 時間為排序，最慢的在最上面：

```bash
$ grep -h "<testcase" `find . -iname "TEST-*.xml"` | sed 's/<testcase time="\(.*\)" classname="\(.*\)" name="\(.*\)".*/\1\t\2.\3/' | sort -rn | head
```
