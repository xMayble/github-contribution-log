Contribution Log — Issue #212: Add $isNumber Compatibility Tests


Adding second-pass compatibility test coverage for the $isNumber expression operator in the DocumentDB functional test suite.



Contribution Number: 1

Student: Mehbub Rohit

Issue: documentdb/functional-tests#212

Status: Phase III Complete

Branch: fix-issue-212


Why I Chose This Issue

I chose this issue because it's well scoped and has a clear definition of done: add compatibility test coverage for the $isNumber expression operator. It's part of a larger effort (#19) to cover the remaining second-pass expression operators, so there are around 20 sibling issues with completed examples I can model my work after instead of starting from scratch. That makes it a realistic first contribution rather than an open-ended one.

It also matches where I am right now as a contributor. The project is in Python, which I'm comfortable working in, and the task centers on writing tests rather than altering core behavior, so I can put my energy into learning the contribution workflow end to end (forking, branching, opening a PR, and responding to maintainer feedback) without also wrestling with an unfamiliar system. What I'm hoping to take away is a feel for how a real project structures its test suite and what its review process is actually like.


Understanding the Issue

Problem Description

The $isNumber expression operator has only a smoke test (test_smoke_isNumber.py). A comparable operator like $toDate has three test files plus a utils folder. Issue #212 asks for second-pass coverage: comprehensive tests beyond the smoke test.

Expected behavior: Full test coverage for all numeric types, all non-numeric types, null, missing, nested expressions, and use in pipeline stages beyond $project.

Current behavior: One smoke test covering only int → true and string → false.

Affected component: documentdb_tests/compatibility/tests/core/operator/expressions/type/isNumber/


Reproduction Process

Environment Setup


OS: Windows 11
Python: 3.13.0 (system install)
Shell: PowerShell
Setup errors: None. The project's venv was already initialized.


Steps to Reproduce


Navigate to documentdb_tests/compatibility/tests/core/operator/expressions/type/isNumber/
Compare to the sibling toDate/ folder. It has test_smoke_toDate.py, test_toDate_basic.py, test_toDate_expressions.py, and a utils/ folder.
isNumber/ contains only test_smoke_isNumber.py. The second-pass coverage is missing.



Observed result: Only 1 test exists for $isNumber. No coverage for numeric types, non-numeric types, null, or missing fields.



Reproduction Evidence

No commit needed. The gap is the absence of files.


Directory contents: isNumber/ contains only test_smoke_isNumber.py
Findings: The smoke test only checks int → true and string → false. All other BSON types (int64, double, Decimal128, bool, null, array, object, ObjectId, and so on) have no test coverage.



Solution Approach

Analysis

The problem is not a bug but a coverage gap. The $isNumber operator was given a smoke test in the first pass, but second-pass coverage was never added.

Proposed Solution

Add three new test files to the isNumber/ folder covering all numeric BSON types that return true, all non-numeric types that return false, and null/missing field behavior.

Implementation Plan (UMPIRE)

Understand. $isNumber returns true for int32, int64, double, and Decimal128. It returns false for all other types including null and missing fields. Only a smoke test exists; full type coverage is missing.

Match. The $and operator tests are the closest pattern. They use ExpressionTestCase, execute_expression, execute_expression_with_insert, and assert_expression_result from utils/utils.py.

Plan.


Create test_isNumber_numeric_types.py for parametrized tests: int32, int64, double, Decimal128 → true
Create test_isNumber_non_numeric_types.py for parametrized tests: string, bool, array, object, ObjectId, Date, and so on → false
Create test_isNumber_null_missing.py for null → false, missing field → false


Implement. Branch: xMayble/functional-tests/tree/fix-issue-212

Review. Followed CONTRIBUTING.md rules: docstrings on every test function, one assertion per test, execute_command wrappers only. Ran --collect-only to confirm the format validator passes before committing.

Evaluate. Run pytest documentdb_tests/compatibility/tests/core/operator/expressions/type/isNumber/ -v against a MongoDB instance. All 87 tests should pass, including the original smoke test.


Testing Strategy

Unit Tests


Done. test_isNumber_numeric_types.py: 48 parametrized tests covering int32, int64, double, Decimal128 as both literals and field references
Done. test_isNumber_non_numeric_types.py: 34 parametrized tests covering string, bool, array, object, ObjectId, Date, Timestamp, Binary, Regex, MinKey, MaxKey, Code
Done. test_isNumber_null_missing.py: 4 tests for null literal, null-valued field, missing field, nested missing path


Integration Tests


Done. All tests run through the aggregation pipeline using execute_expression and execute_expression_with_insert
Done. Original smoke test still collects and passes format validation (regression check)


Manual Testing

Ran pytest --collect-only against the isNumber/ folder. All 87 tests collected with zero format validation errors. The project's validator enforces docstrings, one assertion per test, and proper use of execute_command wrappers. All passed.


Implementation Notes

Week 1 Progress

Built all three test files in one session. The main challenge was understanding how the project structures parametrized tests. I spent time reading the $and and $toDate operator tests to understand the ExpressionTestCase and BaseTestCase patterns before writing anything. Once the pattern was clear, implementing the tests was straightforward.

Code Changes

Files added under documentdb_tests/compatibility/tests/core/operator/expressions/type/isNumber/:


test_isNumber_numeric_types.py
test_isNumber_non_numeric_types.py
test_isNumber_null_missing.py


Key commit: d8de588 — "Add $isNumber compatibility tests for numeric types, non-numeric types, null, and missing fields"

Branch: fix-issue-212

Approach Decisions

Used a custom IsNumberTest dataclass extending BaseTestCase with a value field, allowing each test case to drive both a literal test and a field-reference test without duplicating the test data.


Pull Request


PR link: To be submitted in Phase IV
PR description: To be written in Phase IV
Maintainer feedback: Pending
Status: Awaiting PR submission



Learnings & Reflections

Technical Skills Gained

Learned how real open source projects structure parametrized test suites using dataclasses instead of anonymous tuples. Understood how the project's format validator enforces test quality at collection time rather than runtime.

Challenges Overcome

Reading through multiple sibling operator tests ($and, $toDate) to identify the right pattern before writing code. The project has several overlapping utility functions and it wasn't immediately obvious which one to use.

What I'd Do Differently Next Time

Read the existing tests in the same folder first before looking at unrelated operators. The $and tests ended up being a closer match than $toDate, which I initially assumed would be the right model.
