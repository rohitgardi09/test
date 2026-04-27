Topic - Capture Exception and Store in DB

# Description

We need to create a exception log mechanism which can catch all type of exceptions in the system and log into the db with detailed stacktrace where we can analysis which class method, code line has thrown the exception.

# Data Flow

1. User / Scheduler / kafka will triger java methods
2. Java method will execute the code flow
3. Code flow may throw exceptions in between
4. We need to catch and log that exception 

# Testing Check List

* Testing which covers all main functionalities

# Developer Checklist

* Class Flow

![ClassFlow.png](/-/project/22/uploads/fb6a534f1bf8fe74d293c775fc49de14/ClassFlow.png)

* 90% Unit test case coverage

# Development Steps

1. Create ExeptionLog Service
2. Create log exception method which send Exception object, Mid, String remark(optional)
3. Inject this class in calling classes
4. Call log exception method in every catch block

# Database Tables

* Exception_Log (Append Only table)

  | Column | Type |
  |--------|------|
  | Type | varchar |
  | Stacktrace | varchar |
  | Path | varchar |
  | Merchant_Id | varchar |
  | Remark | varchar |
  | Created_At | number |
  | Created_By | varchar |


# Acceptance Criteria

* Test cases should have 90% coverage

