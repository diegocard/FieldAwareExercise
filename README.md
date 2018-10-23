## Requirements

#### Part 1
Write a program in javascript that reads the log entries from a file and stores them in appropriate data structures. The data should be stored in memory, ie. only use standard in memory data structures, not an external database. We would like to see demonstrations of good practice: e.g. good variable and function naming, code documentation and unit tests.

#### Part 2
Using the code from (1) write functions that:
* returns all log lines with a given log level
* returns all log lines belonging to a given business
* returns all log lines for a given session id
* returns all log lines within a given date range 

#### Part 3
Write a profiling class that can be used to collect basic performance statistics about different functions. The class should calculate the time taken for a function to execute. The class should store the execution times for each decorated function in a dictionary and calculate the maximum, minimum and average execution times. At the end of a profiling run print out a test report for the profiled functions.

## Design guidelines

I have implemented a guidelines for the design of the solution. Namely:
* The solution should be built as if it were to run on NetSuite.
  - Use ES5 only (No ES6, ES7 or transpiling since these solutions do not translate well into NetSuite)
  - Structure the solution into AMD modules (as opposed to classes or CommonJS modules).
  - RequireJS was chosen as a module manager since it closely resembles the structure of SuiteScript 2.0 modules
* Implement a highly efficient solution that could scale well
* Focus on readability over brevity - lots of comments
* Abstract all reusable components into independent modules
* Implement a robust unit testing suite, using Mocha and Chai

## Repo structure

- [Logs](https://github.com/diegocard/FieldAwareExercise/tree/master/Logs): Contains the LogParser and LogProcessor modules which parse and process logs respectively for parts 1 and 2. Also contains tests for these modules.
- [Performance](https://github.com/diegocard/FieldAwareExercise/tree/master/Performance): Contains the PerfDecorator module for part 3. Also contains tests for this module.
- [Utils](https://github.com/diegocard/FieldAwareExercise/tree/master/Utils): Contains the IndexedDataStorage module which is a reusable efficient in-memory data structure used for parts 1 and 2, and the JSUtils module which contains reusable utilities. Also contains tests for these modules.
- [README.md](https://github.com/diegocard/FieldAwareExercise/blob/master/README.md): The documentation you are currently reading.

## Implementation notes

### Parts 1 and 2

* I wanted to use an efficient in-memory structure that could scale well. Searching logs by log level, business or session ID is resolved in ```O(1)``` consequently. Searching logs by a given date range is resolved in ```O(n)```.
* I abstracted the aforementioned memory structure into a module called  ```IndexedDataStorage```. Not only this makes the implementation of the data structure reusable, but also very easy to extend. Adding a new index (say by request ID) is trivial now.
* Note that building indexes in ```IndexedDataStorage``` does not increment memory complexity as indexes only contain references to the log objects. This means that there is only a single in-memory object for every log.
* The log parser was built into a specific module, abstracting the structure of logs and making it easy to extend.

### Part 3

* There are multiple ways to implement wrappers or decorators in JavaScript. By far the best approach would be to use ES6/7 function decorators (similar to decorators on Java or Python). However, these are not available in NetSuite environments and require transpiling. Therefore I opted for a more traditional implementation of the decorator pattern that works everywhere.
* One approach to calculating min, max and average execution times is to store all executions of the given function. However, this does not scale well.
* Imagine that you execute the function 1000 times. That would mean creating a 1000-length array and iterating through it constantly to calculate averages.
* In order to provide an efficient implementation I used a bit of math and implemented a rolling average. This means that only the max, min, current average and number of executions are stored at any given time (only 4 numbers). This is a very efficient implementation that scales well - performance is not affected by the amouynt of times the function needs to execute.

## Usage

### Parts 1 and 2

```js
require(["Logs/LogProcessor"], function(LogProcessor) {
    // Test data provided for the exercise
    var data = 
    `2012-09-13 16:04:22 DEBUG SID:34523 BID:1329 RID:65d33 'Starting new session'
    2012-09-13 16:04:30 DEBUG SID:34523 BID:1329 RID:54f22 'Authenticating User'
    2012-09-13 16:05:30 DEBUG SID:42111 BID:319 RID:65a23 'Starting new session'
    2012-09-13 16:04:50 ERROR SID:34523 BID:1329 RID:54ff3 'Missing Authentication token'
    2012-09-13 16:05:31 DEBUG SID:42111 BID:319 RID:86472 'Authenticating User'
    2012-09-13 16:05:31 DEBUG SID:42111 BID:319 RID:7a323 'Deleting asset with ID 543234'
    2012-09-13 16:05:32 WARN SID:42111 BID:319 RID:7a323 'Invalid asset ID'`;

    var lp = new LogProcessor();
    lp.processInput(data);

    // Get all log lines with a given log level
    lp.getLogsByLogLevel("WARN");
    /*
        {
            "description":"Invalid asset ID",
            "date":"2012-09-13T19:05:32.000Z",
            "loglevel":"WARN",
            "sid":"42111",
            "bid":"319",
            "rid":"7a323"
        }
    */ 

    // Get all log lines belonging to a given business
    lp.getLogsByBusiness("1329");

    // Get all log lines for a given session id
    lp.getLogsBySession("319");

    // Get all log lines within a given date range
    lp.getLogsByDateRange("2012-09-13 16:04:22", "2012-09-13 16:04:50");
});
```

### Part 3

```js
require(["Performance/PerfDecorator"], function(PerfDecorator) {
    
    // Creates a function that executes in a random time between minDelay and maxDelay
    function createDelayedFunction(minDelay, maxDelay) {
        return function delayedFunction() {
            var delay = Math.floor(Math.random()*(maxDelay-minDelay+1)+minDelay);
            var start = new Date().getTime();
            while ((new Date() - start) < delay) {}
            return 1;
        }
    }

    // Create a function that takes in between 50 to 100 ms to execute
    var decoratedFunc = PerfDecorator(createDelayedFunction(50, 100));

    // Execute the function 10 times
    for (var i=0; i<10; i++) {
        decoratedFunc();
    }

    // Get min, average and max execution times
    var minTime = decoratedFunc.Min;
    var avgTime = decoratedFunc.Average;
    var maxTime = decoratedFunc.Max;

    // Read the amount of executions
    var numSamples = decoratedFunc.NumSamples;

    // Print all log information
    decoratedFunc.logPerfInfo();
});
```

## Tests

* [Click here](https://diegocard.com/FieldAwareExercise/Logs/LogTests.html) to run the test suite for logging-related modules (part 1 and 2).
* [Click here](https://diegocard.com/FieldAwareExercise/Performance/PerfTests.html) to run the test suite for the performance decorator module (part 3).
* [Click here](https://diegocard.com/FieldAwareExercise/Utils/UtilsTests.html) to run the test suite for utility methods.

