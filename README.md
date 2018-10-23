# FieldAware Exercise

## Requirements

1) Write a program in javascript that reads the log entries from a file and stores them in appropriate data structures. The data should be stored in memory, ie. only use standard in memory data structures, not an external database. We would like to see demonstrations of good practice: e.g. good variable and function naming, code documentation and unit tests.

2) Using the code from (1) write functions that:
* returns all log lines with a given log level
* returns all log lines belonging to a given business
* returns all log lines for a given session id
* returns all log lines within a given date range 

3) Write a profiling class that can be used to collect basic performance statistics about different functions. The class should calculate the time taken for a function to execute. The class should store the execution times for each decorated function in a dictionary and calculate the maximum, minimum and average execution times. At the end of a profiling run print out a test report for the profiled functions.
