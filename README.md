# Bears-Benchmark

This is for CS527 Software Engineer focusing on software testing

## Milestone 2

Bears (https://github.com/bears-bugs/bears-benchmark): 20 randomly chosen bugs. Please refer to the JSON file with the information of all bugIds and branches and randomly select 20 from them. There will be 4 commits available in each branch. You can check out the commits corresponding to each bug's buggy and patched versions.

### Step1: For each branch representing each project, check out:
```bash
python3 scripts/checkout_bug.py --bugId Bears-1

```

### Step2: Run the given script
```bash
python3 scripts/run_tests_bug.py --bugId Bears-1 # this should fail because the checkout_bug.py has checked out the third, buggy commit
```

### Step3: The above script will generate failed/error tests:


```bash
Tests run: 9, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 0.007 sec <<< FAILURE! - in com.fasterxml.jackson.databind.filter.ProblemHandlerTest
testWeirdStringHandling(com.fasterxml.jackson.databind.filter.ProblemHandlerTest)  Time elapsed: 0.004 sec  <<< ERROR!

Results :
Tests in error: 
  ProblemHandlerTest.testWeirdStringHandling:247 » InvalidFormat Can not deseria...
```

### Step4: If you run the following command to run the error test, the build should fail:
```bash
mvn -Dtest=ProblemHandlerTest#testWeirdStringHandling test
```
But if you go the patched version and run the same command again, build should succeed, meaning that the bug has been fixed:
```bash
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.fasterxml.jackson.databind.filter.ProblemHandlerTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.103 sec - in com.fasterxml.jackson.databind.filter.ProblemHandlerTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.215 s
[INFO] Finished at: 2024-03-27T19:50:26-05:00
[INFO] ------------------------------------------------------------------------
```

## Milestone 3
Now we have the buggy and patched version, and also the diff file, we know that classes that we change to fix the bug. We can now use tools to autogenerate tests.

### Step1: Go to buggy and patched versions, compile the whole project:
```bash
mvn compile
```

### Step2: Generate the necessary dependencies:
```bash
mvn dependency:build-classpath -Dmdep.outputFile=cp.txt
```

### Step3: Go to diff file and find the classes being modified:
```bash
--- a/bugs/Bears-20/Buggy-Version/src/main/java/com/fasterxml/jackson/databind/deser/std/FromStringDeserializer.java
+++ b/bugs/Bears-20/Patched-Version/src/main/java/com/fasterxml/jackson/databind/deser/std/FromStringDeserializer.java
```

### Step4: Use randoop to auto generate the tests:
```bash
java -classpath "$(cat cp.txt):/Users/jingyaogu/Desktop/randoop-4.3.2/randoop-all-4.3.2.jar:/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-20/Patched-Version/target/classes/" randoop.main.Main gentests --testclass=com.fasterxml.jackson.databind.deser.std.FromStringDeserializer

```

```bash
java -classpath "$(cat cp.txt):/Users/jingyaogu/Desktop/randoop-4.3.2/randoop-all-4.3.2.jar:/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-20/Buggy-Version/target/classes/" randoop.main.Main gentests --testclass=com.fasterxml.jackson.databind.deser.std.FromStringDeserializer

java -classpath "/Users/jingyaogu/Desktop/randoop-4.3.2/randoop-all-4.3.2.jar:/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-168/Buggy-Version/address-controller/target/classes" randoop.main.Main gentests --testclass=io.enmasse.controller.api.DefaultExceptionMapper

java -classpath "/Users/jingyaogu/Desktop/randoop-4.3.2/randoop-all-4.3.2.jar:/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-142/Buggy-Version/modules/activiti-engine/target/classes" randoop.main.Main gentests --testclass=org.activiti.engine.impl.history.handler.ActivityInstanceEndHandler

java -classpath "/Users/jingyaogu/Desktop/randoop-4.3.2/randoop-all-4.3.2.jar:/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-211/Patched-Version/cbor/target/classes" randoop.main.Main gentests --testclass=com.fasterxml.jackson/dataformat/cbor/CBORFactory
Bears/Bears-211/Patched-Version/cbor/target/classes/com/fasterxml/jackson/dataformat/cbor/CBORFactory.class

```

### For patched version:
Results are: regression tests on the patched version that fail on the buggy version. Note that here, we are looking for test failures, NOT test errors. Test errors may happen because a code in the patched version does not exist in the buggy version. 

### For buggy version:
Results are: error revealing tests on the buggy version that pass on the patched version

### Step5: Use Evosuite to auto generate the tests:

#### Run evosuite:
```bash
java -jar /Users/jingyaogu/Desktop/evosuite-1.0.6.jar -class com.fasterxml.jackson.databind.introspect.POJOPropertiesCollector -projectCP "/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-6/Patched-Version/target/classes/:$(cat cp.txt)"

java -jar /Users/jingyaogu/Desktop/evosuite-1.0.6.jar -class com.fasterxml.jackson.databind.introspect.POJOPropertiesCollector -projectCP "/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-6/Buggy-Version/target/classes/:$(cat cp.txt)"

java -jar /Users/jingyaogu/Desktop/evosuite-1.0.6.jar -class org.activiti.engine.impl.history.handler.ActivityInstanceEndHandler -projectCP "/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-142/Patched-Version/target/classes/:$(cat cp.txt)"

java -jar /Users/jingyaogu/Desktop/evosuite-1.0.6.jar -class org.activiti.engine.impl.history.handler.ActivityInstanceEndHandler -projectCP "/Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-142/Patched-Version/modules/activiti-engine/target/classes:$(cat cp.txt)"

```
After running the command, evosuite should generate tests that can be seen under evosuite-test folder. Copy those tests to src/java folders where the test belongs. Then run the following to have the evosuite run time installed.

```bash
mvn install:install-file -Dfile=/Users/jingyaogu/Desktop/evosuite-standalone-runtime-1.0.6.jar -DgroupId=org.evosuite -DartifactId=evosuite-standalone-runtime -Dversion=1.0.6 -Dpackaging=jar
```
Then add dependency to pom.xml

```bash
<dependency>
    <groupId>org.evosuite</groupId>
    <artifactId>evosuite-standalone-runtime</artifactId>
    <version>1.0.6</version>
    <scope>test</scope>
</dependency>
```
Finally run to see the test results
```bash
mvn -Dtest=com.fasterxml.jackson.databind.deser.std.FromStringDeserializer_ESTest test
```

## Milestone 4

### Step1: Find the classes that are being modified from the diff file:

In my case, I have 
```
src/main/java/com/fasterxml/jackson/databind/SerializerProvider.java
SerializerProvider
```

### Step2: Find the corresponding clover report in html for that specific class:

In my case, I am opening the html file here:
```
file:///Users/jingyaogu/Desktop/CS527-Team-15/Bears/Bears-16/Buggy-Version/target/site/clover/com/fasterxml/jackson/databind/SerializerProvider.html#SerializerProvider
```

### Step3: Find the corresponding row number of the statements removed or modified in that class

Also find the total number of failed tests, total number of passed tests, and the Id of failed tests.
```
const totalFail = 5;
const totalPass = 1684;
const failedTestId = 243;

// 创建一个数组用于存储每一行的分数及其行号
const lineScores = [];

for (let i = 3; i <= 1078; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // 如果没有测试覆盖这一行，跳过
    }
    let fails = 0; // 计数器，用于统计失败的测试数量
    let passes = 0; // 计数器，用于统计通过的测试数量
    
    // 遍历覆盖当前行的测试用例
    testsCoveringLine.forEach(test => {
        if (test === failedTestId) {
            fails++; // 如果测试ID与失败的测试ID相匹配，失败计数增加
        } else {
            passes++; // 否则，通过计数增加
        }
    });
    
    // 计算可疑度分数
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness > 0) {
        // 只记录有可疑度分数的行
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// 按可疑度分数降序排序
lineScores.sort((a, b) => b.score - a.score);

// 找到第219行的排名
const rankOfLine219 = lineScores.findIndex(item => item.line === 730) + 1;

// 打印每一行的分数和第219行的排名
console.log("Line Scores and Suspiciousness:");
lineScores.forEach(item => console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}`));
console.log(`Rank of Line 730: ${rankOfLine219}`);
```

























