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
This is for Bears-19
```
const totalFail = 4; // Assuming this still represents the total number of failed tests
const totalPass = 1719; // Total number of passed tests
const failedTestIds = [68, 416, 543, 1339]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 907; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 495;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```

For Bears 6:
```
const totalFail = 5; // Assuming this still represents the total number of failed tests
const totalPass = 1684; // Total number of passed tests
const failedTestIds = [423, 431, 1050, 1494, 1547]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 1078; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 730;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```

For Bears-20:
```
const totalFail = 4; // Assuming this still represents the total number of failed tests
const totalPass = 1734; // Total number of passed tests
const failedTestIds = [1066, 1698]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 310; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 115;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```

For Bears-7:
```
const totalFail = 5; // Assuming this still represents the total number of failed tests
const totalPass = 1687; // Total number of passed tests
const failedTestIds = [327, 1230]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 235; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 170;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```









For Bears-26:
```
const totalFail = 4; // Assuming this still represents the total number of failed tests
const totalPass = 1754; // Total number of passed tests
const failedTestIds = [327, 1230]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 235; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 170;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```

For bears-140
```
const totalFail = 4; // Assuming this still represents the total number of failed tests
const totalPass = 1754; // Total number of passed tests
const failedTestIds = [327, 1230]; // Array of failed test IDs

// Create an array to store the score and line number of each covered line
const lineScores = [];

for (let i = 3; i <= 235; i++) {
    var testsCoveringLine = clover.srcFileLines[i];
    if (testsCoveringLine.length === 0) {
        continue; // If no tests cover this line, skip it
    }
    let fails = 0; // Counter for the number of failing tests
    let passes = 0; // Counter for the number of passing tests
    
    // Loop through tests covering the current line
    testsCoveringLine.forEach(test => {
        if (failedTestIds.includes(test)) {
            fails++; // Increment fail counter if test ID is in the list of failed test IDs
        } else {
            passes++; // Otherwise, increment pass counter
        }
    });
    
    // Calculate the suspiciousness score
    const suspiciousness = fails / totalFail / (fails / totalFail + passes / totalPass);
    
    if (suspiciousness >= 0) { // Only record lines with a non-zero suspiciousness score
        lineScores.push({ line: i, score: suspiciousness });
    }
}

// Sort by suspiciousness score in descending order
lineScores.sort((a, b) => b.score - a.score);

// Assign ranks considering ties, only for non-zero scores
let currentRank = 1;
let previousScore = -1; // Initialize to a non-valid score
let countOfLinesAbove = 0;

lineScores.forEach((item, index) => {
    if (item.score === previousScore) {
        // Same score as previous, assign the same rank
        item.rank = currentRank;
    } else {
        // New score, update rank based on the actual index of non-zero scores
        currentRank = countOfLinesAbove + 1;
        item.rank = currentRank;
        previousScore = item.score;
    }
    countOfLinesAbove++;
});

// Find the entry for Line 1181 and print its rank
const lineOfInterest = 170;
const entry = lineScores.find(item => item.line === lineOfInterest);

// Print each line's score, suspiciousness, and rank for non-zero scores
console.log("Line Scores, Suspiciousness, and Ranks:");
lineScores.forEach(item => {
    if (item.score > 0) {
        console.log(`Line ${item.line}: Suspiciousness Score: ${item.score.toFixed(4)}, Rank: ${item.rank}`);
    }
});

// Print the rank of Line 1181
if (entry) {
    console.log(`Rank of Line ${lineOfInterest}: ${entry.rank}`);
} else {
    console.log(`Line ${lineOfInterest} not found in the coverage data.`);
}

// Count how many scores have non-zero suspiciousness
const nonZeroSuspiciousCount = lineScores.filter(item => item.score > 0).length;

// Print the count of non-zero suspiciousness
console.log(`Number of lines with non-zero suspiciousness: ${nonZeroSuspiciousCount}`);
```































