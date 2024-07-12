**Key points:**



* Tets derive value from the trust engineers place in them
* Bad tests suites can be worse than no test suites at all
* Tests measured in size and scope (size being a single process, a single machine, or multiple machines, scope being a single method or class, a few components, or the whole enchilada)
* THEY STRONGLY DISCOURAGE THE USE OF CONTROL FLOW STATEMENTS LIKE CONDITIONALS AND LOOPS IN A TEST
* Test everything you want to work (“if you like it you should’ve put a test on it”)
* **Tests need to be treated like production code **
* **You need to incentivize engineers to care about their tests. Reward them as much for having rock-solid tests as you would for having a great feature launch**
* Changing the testing culture in organizations take time

---------


# Testing Overview


## Why do we write tests?



* Simplest test is designed by 
    * Single behavior -- method or API you’re calling
    * A specific input with some value you pass to API
    * Observable output or behavior
    * A controlled environment like a single isolated process 
* A hundred or thousand of these are a test suite 
* Instability and slowness grow as the test suite grows too 
* **Tests derive value from the trust engineers place in them**
    * If testing becomes a productivity sink, constantly inducing toil and uncertainty, engineers will lose trust and begin to find workarounds
        * ** Bad test suite can be worse than no test suite at all**


### The Story of Google Web Server



* “To address these problems, the tech lead (TL) of GWS decided to institute a policy of

engineer-driven, automated testing. As part of this policy, all new code changes were

required to include tests, and those tests would be run continuously. Within a year of

instituting this policy, the number of emergency pushes **dropped by half.”**



* One of the key insights the GWS experience taught us was that yo**u can’t rely on pro‐**

    **grammar ability alone to avoid product defects.** Even if each engineer writes only the


    occasional bug, after you have enough people working on the same project, you will


    be swamped by the ever-growing list of defects.** Imagine a hypothetical 100-person**


    **team whose engineers are so good that they each write only a single bug a month.**


    **Collectively, this group of amazing engineers still produces five new bugs every workday**. Worse yet, in a complex system, fixing one bug can often cause another, as engineers adapt to known bugs and code around them



## Designing a test suite



* Google learned early on is engineers favored writing larger, system-scale tests, but these tests were slower, less reliable, and more difficult to debug than smaller tests
* Google learned that more small tests are better, but what is “small”? 
* There are two dimensions for every test case: size and scope
    * Size is the resources that are required to run a test case --- memory, processes, time
    * Scope refers to the specific code paths we’re verifying. 
    * Note: executing a line of code is different from verifying that it worked as expected. Size and scope are interrelated but distinct concepts 


### Test Size



* Google classifies every test into a size and encourage engineers to always write the smallest possible test for a given piece of functionality 
    * Test size is determined by how it runs, what it’s allowed to do, and how many resources it consumes
* Small tests run on a single process, medium tests run on a single machine, and large tests run wherever they want. 


![image](https://github.com/user-attachments/assets/c3fbe8d1-9617-4c4b-8e9c-c49684e8b5fe)





* They use this distinction rather than the traditional “unit” or “integration” because the most important qualities they wnt from their test suites are speed and determinism, regardless of scope of test
    * Small tests are almost always faster and more deterministic than tests that involve more infrastructure or consume or resources 
    * Placing restrictions on small tests makes speed and determinism easier to achieve 
    * Note:[ The determinism of a system, or a function, is the property that by feeding the same input to the system multiple times, the outputs are identical.](https://medium.com/@wurui90/how-to-make-a-complex-software-deterministic-fd06d135a416)


#### Small Tests



* Must run in a single process 
    * In many languages, they restrict this further to say that the test must run on a single thread
    * THis means **the code performing the test must run in the same process as the code being tested. **You can’t run a server and have a separate test process connect to it. It also means you can’t run a third-party program like a database as part of your test 
* The other important constraint on small tests is that they aren’t allowed to sleep, perform I/O operations, or make other blocking calls
    * This means small tests aren’t allowed to access network or disk.
* The purpose of these restrictions is to ensure that small tests don’t have access to the main sources of test slowness or nondeterminism. A test that runs on a single process and never makes blocking calls can effectively run as fast as the CPU can handle. 


#### Medium Tests



* Can span multiple processes, use threads, and make blocking class, including network calls, to `localhost`
* The only remaining restriction is that network calls can only be made to `localhost`, e.g. test must run on a single machine 
* More possibilities here
    * You can run a db instance to validate that the code you’re testing integrates correctly in a more realistic setting
    * Can test a combo of web Ui and server code -- this usually involves tools liek WebDriver (or playwright!) that starts a real browser and controls it remotely via the test process 


#### Large tests 



* True e2e, often run only during the build and release process so as not to impact dev workflow


#### Aspects common to all test sizes



* Should be hermetic: a test should contain all of the info necessary to set up, execute, and tear down its env
* Tests should assume as little as possible about the outside environment, such as the order in which the tests are run (e.g. they shouldn’t assume a shared database
* **THEY STRONGLY DISCOURAGE THE USE OF CONTROL FLOW STATEMENTS LIKE CONDITIONALS AND LOOPS IN A TEST**
    * More complex tests flows risk containing bugs themselves and make it more difficult to determine the cause of a test failure 


### Test Scope



* Narrow scoped tests (commonly called unit tests) are designed to validate logic in a small, focused part of the codebase, like an individual class or method 
* Medium scope dests (commonly called integration tests) are designed to verify interactions between a small number of components 



![image](https://github.com/user-attachments/assets/724ef041-7e17-4f0c-9810-fb966f361492)



#### Testing Antipatterns

![image](https://github.com/user-attachments/assets/6ef864d0-a641-4573-9118-b2028c2e1c86)

* The ice cream cone yields suites that are slow, unreliable, and difficult to work with. **This pattern often appears in projects that start as prototypes and are quickly rushed to production, never stopping to address testing debt**
* Hourglass pattern occurs when tight coupling makes it difficult to instantiate invidivdual dependencies in isolation


### The Beyonce Rule



* Which behaviors or properties actually need to be tested? The straightforward answer is: test everything that you don’t want to break
    * Performance
    * Behavior correctness
    * Accessibility
    * Security
    * Also less obvious stuff, like how a system handles failure
* “If you liked it then you shoulda put a test on it”
* **The Beyonce Rule is often invoked by infrastructure teams that are responsible for making changes across the entire codebase. **If unrelated infrastructure changes pass all of your tests but still break your team’s product, you’re on the hook for fixing it an adding the additional tests 


### Testing for Failure



* Instead of waiting for failure, write automated tests that simulate common kinds of failures: exceptions, RPC errors, latency in integration and e2e tests
* Also Chaos engineering


## Testing at Google Scale



* They use a monorepo (we don’t) so disregard
* They also do trunk based development. All changes are committed to the repo head and are immediately visible for everyone to see (???)
* When building a product or service, Google builds not only the main code but also nearly all the dependencies from their source code, ensuring consistency and control over the entire software stack.
* They do this via CI; a key component off their CI system is their Test Automated Platform (TAP)


### The pitfalls of a large test suite



* Brittle tests, which over-specify expected outcomes or rely on extensive and complicated boilerplate, can actually rest change. These poorly written tests can fail even when unrelated changes are made 
    * E.g.: you make a five-line change to a feature and find dozes of unrelated, broken tests. This is horrible DevX
* Some of the worst offenders of brittle tests come from misuse of mock objects. Understanding the limitations of mock objects can help you avoid misusing them
    * (more on this in chapter 13)
* Larger tests can also be slower, which is also horrible dev x. 
* The secret to living with a large test suite is to treat it with respect, e.g. **treat it like production code**. When simple changes begin taking nontrivial time, spend effort making tests less brittle. 
    * **You need to incentivize engineers to care about their tests. Reward them as much for having rock-solid tests as you would for having a great feature launch**
    * Set appropriate performance goals and refactor slow or marginal tests
* Additional, invest in testing infra by developing linters, documentation, and other assistance that makes it more difficult to write bad tests 
