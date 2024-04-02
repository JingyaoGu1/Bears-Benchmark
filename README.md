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























