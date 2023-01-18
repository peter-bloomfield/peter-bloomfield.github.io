---
layout: post
title: How to write a good unit test
date: '2020-06-28 21:23:13'
tags:
- testing
- best-practices
- unit-tests
- guidelines
redirect_from:
- /how-to-write-a-good-unit-test
---

When used correctly, [unit testing](https://en.wikipedia.org/wiki/Unit_testing) can be valuable for ensuring the long-term quality and correctness of your code. However, it's a skill which can take time and discipline to do well, even if you're already an adept programmer. In this post, I'll outline some of the key guidelines which I've found helpful for writing good unit tests.

## BRIEF

A good unit test should be BRIEF:

1. Brief
2. Reliable
3. Independent
4. Explicit
5. Focused

## 1. Brief

An individual unit test should be brief in two ways: the test code should be **short** and it should execute **quickly**.

### 1.1. Short

Your unit test shouldn't involve hundreds of lines of code. As with any other area of programming, excessively long code is generally over-complicated, hard to read, and hard to maintain. This can cause mistakes and oversights, reducing the value of the test.

Aim to keep each test as concise as possible, without sacrificing readability. As a rule-of-thumb, I would suggest a soft limit of 20 lines, but ideally fewer. It depends on what you're testing though, and how complex the main code is.

If a unit test is getting quite long, first consider if you could split it up into multiple tests. Several short tests are often better than one long test. You may also be able to move parts of the code out into separate functions which could be reused by other tests.

Also consider using setup/tear-down functions, [test fixtures](http://sqa.fyicenter.com/FAQ/JUnit/What_Is_a_JUnit_Test_Fixture_.html), and [paramaterised tests](https://www.tutorialspoint.com/junit/junit_parameterized_test.htm), if your test framework supports them.

### 1.2. Quick

Most unit tests should take no more than a few milliseconds each to run. A complete set of hundreds or thousands of tests for a whole component should ideally only take a few seconds if you have a reasonably powerful workstation.

If they're taking several minutes or longer then they can become a drain on productivity. Programmers will be less likely to run them regularly and consequently less likely to work on them. This ultimately reduces the value of the tests.

If a test is particularly slow then try to identify what's causing the delay. There will often be ways to reduce or avoid it. For example, if it takes a while to setup the data for a test then see if you could load predefined data from disk. If the test needs to wait for an external resource then perhaps you could replace the resource (or its interface) with a [mock or stub](https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing) etc.

Unfortunately, it's not always possible to avoid having some slow tests. Sometimes, you simply have to test something which takes a while.

An easy way to improve overall execution time may be to run the tests in parallel. This can mean several short tests can be executed in one thread while a longer test executes in another. Some test frameworks support this directly, while others can be made to run in parallel via other tools. However, be careful if your tests have any side-effects which might interfere with each other, such as using system resources or writing to a file on disk.

## 2. Reliable

This is one of the most important guidelines: A unit test must give a consistent result no matter how many times you run it. If a test sometimes fails randomly when it should pass (or sometimes passes when it should fail) then you simply can't trust the result either way, and you potentially can't trust any subsequent changes to it.

This is particularly important if have a build server or continuous integration pipeline which runs unit tests automatically. A spurious failure might result in wasted time spent investigating the problem. Eventually, failure-fatigue means somebody might decide to ignore the results entirely, potentially forcing through a build which has a genuine problem.

A common cause of unreliability is having tests which depend on (or otherwise affect) each other. This must be avoided as you can't guarantee that they will always run in the same order. I will go into more detail on this in the next section. In short, each test should be completely standalone.

You should also ensure that the tests aren't relying on any assumptions about the system or environment they are running on. For example, don't assume that the tests will be executed from a particular location on disk, or that the Operating System will be a specific version.

Another potential cause of unreliable unit tests is timing. If your test is waiting for something to finish before checking the result then it needs to be confident about how long it will take. Try to avoid making assumptions about the duration as timings can vary, especially if the system is under variable load. My preferred approach (where possible) is to keep checking for the result repeatedly, several times per second. If the result hasn't been received within a very large timeout then the test will fail.

## 3. Independent

Unit tests need two kinds of independence: internal and external.

### 3.1. Internally independent

Unit tests should be independent of each other. This means you should get the same results no matter what order you execute the tests in. It should also be possible to execute a single unit test on its own.

A common beginner mistake is to share an object or other resource between tests. If one test changes the state then the behaviour of a subsequent test may be affected. This can result in unreliable tests if the framework doesn't execute them in the same order every time. It can also make the tests difficult to maintain and modify in future as the inter-dependencies may not be obvious.

In general, the safest approach is for each test to have its own copy of anything it needs to work on. If shared state is unavoidable for some reason, then be sure to clear it between each test. Some unit test frameworks facilitate this by letting you specify a setup function to be called before each test. For example, see [`beforeEach()` in Jest](https://jestjs.io/docs/en/setup-teardown).

With that said, it may be OK to share immutable things between tests; i.e. objects or data which cannot be changed. This can be useful if the setup process is computationally expensive. However, if you are going to run tests in parallel then you may need to ensure that read access is thread-safe in some languages.

In some areas of programming, such as desktop development, unit tests might affect local resources such as files on disk. In this case, independence between tests may also involve ensuring each test runs in a separate temporary directory. This can avoid potential file conflicts even when the tests are executed in parallel.

### 3.2. Externally independent

Where possible, a unit test (including the code it is testing) shouldn't rely on the state or availability of any separate system or resource. This is because an external dependency can make the test unexpectedly slow or unreliable.

A common mistake is testing code which makes a web-request. If the external server is offline, unreachable, or misconfigured, then the test is likely to fail, despite there being nothing wrong with the code being tested.

There are a few potential solutions to this. The simplest is often to refactor the code being tested so that you can test the request generation and response handling in isolation, without any request actually being sent. [Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) can be useful here as it can let you provide a mock or fake object to intercept the request.

Alternatively, if you can't refactor the code being tested then you could run a temporary server as part of the unit tests. You would ensure the request is directed there instead of the external server. This would allow you to check that the expected request is received and that various responses are handled correctly.

Similar principles may apply to dealing with other external dependencies. However, not all such dependencies can be avoided in all cases. If reliability becomes an issue then it may be better to remove the unit test entirely, relying on coverage from system or acceptance testing instead.

## 4. Explicit

Another programmer reading your code should be able to see clearly what each unit test is doing. The most important part of this is giving each test a descriptive name. It should identify the piece of code being tested and the expected behaviour. Don't be afraid of ending up with long names.

For example, let's say I have a class called `Widget` which has a member function called `setName()`. I want to test that it throws an exception if the name passed to it is empty. If I was writing a test for this in C++ with [GoogleTest](https://github.com/google/googletest) then it might look like this:

```c++
TEST(Widget, setNameThrowsInvalidArgumentIfSpecifiedNameIsEmpty)
{
    // ...
}
```

Alternatively, if I was writing it in JavaScript with the [Jest](https://jestjs.io/) framework then it might look like this:

```javascript
describe('Widget', () => {
    describe('setName', () => {
        test('throws InvalidArgument if specified name is empty', () => {
            // ...
        });
    });
});
```

These are just intended as examples of how I like to structure test names. There are many valid ways to do it. Different frameworks will also have different conventions and restrictions. The important thing is that you include the essential information in some way. Ideally, you want the tests to be able to act like a form of documentation.

It's also important to be consistent. This can make it easier to search the tests, browse the test report, and filter tests by name.

Another aspect of making tests explicit is ensuring that your test code is readable. For example, follow good code formatting conventions and use meaningful variable names. This is similar to making any other code readable so it can be a good idea to include your test code in any static analysis and linting which you may already apply to the main code.

Using explicit test data can also be helpful. For example, your assertions could compare against hard-coded expected values where possible rather than using values computed at runtime. This can make it easier to manually verify that the test is operating correctly.

When a failure happens, the test output should be readable as well so that the problem can be diagnosed. It isn't always possible to control this directly though, depending on your framework. I would suggest deliberately making some tests fail so that you can experiment with it. If anything is unclear then you may be able to adjust which assertions you use, or include additional descriptive information in the output to indicate what went wrong.

## 5. Focused

Each unit test should only test one thing; e.g. one behaviour or condition. This helps ensure that the test code is clean and self-contained. It can also help ensure that you can pin-point the problem very quickly when a test fails.

This doesn't necessarily mean that each test should only have one assertion. For example, sometimes you may need to validate multiple related pieces of data returned by a single function call. However, you should be cautious about using more than one, and consider whether it would be appropriate to have a separate test.

As with any other code, if a test is doing too many things then it becomes difficult to read, modify, and maintain. Additionally, combining multiple test cases can mean something important gets overlooked.

If you're struggling to keep a test focused then it could be a sign that the code being tested was written poorly. Unit testing in general is much easier if you've been following good software engineering principles like [KISS](https://en.wikipedia.org/wiki/KISS_principle), [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), and [SOLID](https://en.wikipedia.org/wiki/SOLID).

## In practice...

### Why do I need unit tests?

It's common for unit testing to be dismissed because it requires more development time without seeming to add any customer-facing value. However, the fact is that everybody makes mistakes, no matter how much experience they have.

As a general rule, the earlier you find a bug, the quicker and cheaper it is to fix. Unit tests are the first line of defence, allowing a programmer to check their own work before it affects anyone else. They also allow an automated system to provide some confidence that existing code has not been broken when new changes are introduced.

As such, unit testing can be a relatively small investment of time now which could prevent much bigger costs in future.

It's also worth noting that unit tests and code reviews are complementary, and I would always advise using both. During a review, other programmers can examine the unit tests to ensure they are written properly and cover suitable test cases.

### When should I write unit tests?

Always write unit tests alongside the main code (i.e. the code being tested). This is important because it can help you to think through the design of your code more carefully, ensuring that it's testable. Retrofitting unit tests to existing code tends to be more difficult and time-consuming, and you are more likely to overlook edge cases.

### When should I run unit tests?

Run the unit tests whenever you make a significant code change. This can help you to catch problems sooner. You don't necessarily need to run the complete set of all tests for every change; the test framework will usually allow you to filter them so that it only runs the tests which are relevant to the part you're working on.

With some frameworks and languages, it's also possible to run the tests automatically whenever a file change is detected. This can be a helpful time-saver.

You should certainly run all the tests before pushing or otherwise uploading your code to ensure you haven't introduced any regressions in existing code. Additionally, I strongly recommend running all the tests regularly on an automated build server. If any test fails, it should stop the build and notify you.

### Does code coverage matter?

Code coverage measures how much of the main code is executed by all of the unit tests combined. If the number is low (e.g. 50%) then it's an indication that not enough of the code is being tested. However, a high number doesn't necessarily mean your tests are good. You could achieve 100% coverage without actually testing for anything meaningful.

Not all code can be unit tested. For example, error handling can be tricky as not all errors can be simulated. As such, I would expect to be able to achieve 80-90% coverage overall in most projects, but I wouldn't take the number too seriously. You really need to look through the individual tests to get a good sense of how thorough the tests are.

## Conclusion

In this post, I've outlined some guidelines which I believe can be useful for writing good unit tests. They're based on my experience as a software engineer in a number of jobs, mainly covering desktop development in C++, back-end web-development in JavaScript and PHP, and some general Python scripting.

I don't suggest being dogmatic about the guidelines though. The reality of being a programmer is that you have to balance time and quality. However, I would usually recommend writing a few high quality tests rather than lots of low quality ones.

I think the most important advice I can give about unit testing is that it's a skill you have to improve with practice. As with programming in general, it's not something you can learn just by reading about it. You have to do it regularly and develop good habits, constantly evaluating your effectiveness and learning from your mistakes.

