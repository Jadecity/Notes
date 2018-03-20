1. 只运行某些test

~~~
Google provides --gtest_filter=<test
string> . The format for the test string is a series of wildcard patterns separated by colons (:).
For example, --gtest_filter=* runs all tests while --gtest_filter=SquareRoot* runs only the
SquareRootTest tests. If you want to run only the positive unit tests from SquareRootTest , use --
gtest_filter=SquareRootTest.*-SquareRootTest.Zero* . Note that SquareRootTest.* means
all tests belonging to SquareRootTest , and -SquareRootTest.Zero* means don't run those tests
whose names begin with Zero.
~~~
2. 重复运行

~~~
   If you pass --gtest_repeat=2 --gtest_break_on_failure on the command line, the
same test is repeated twice. If the test fails, the debugger is automatically invoked.
~~~

3. 临时禁用test

~~~
simply add the DISABLE_ prefix to the logical test name or the individual unit test name and it won't execute.
~~~

   ~~~c++
   #include "gtest/gtest.h"
       
   TEST (DISABLE_SquareRootTest, PositiveNos) {
   EXPECT_EQ (18.0, square-root (324.0));
   EXPECT_EQ (25.4, square-root (645.16));
   EXPECT_EQ (50.3321, square-root (2533.310224));
   }
   OR
   TEST (SquareRootTest, DISABLE_PositiveNos) {
   EXPECT_EQ (18.0, square-root (324.0));
   EXPECT_EQ (25.4, square-root (645.16));
   EXPECT_EQ (50.3321, square-root (2533.310224));
   }
   ~~~

   

If you want to continue running the disabled tests, pass the` -gtest_also_run_disabled_tests.`

4. 浮点数的断言

   ~~~c++
   ASSERT_FLOAT_EQ (expected, actual)
   ASSERT_DOUBLE_EQ (expected, actual)
   ASSERT_NEAR (expected, actual, absolute_range)

   EXPECT_FLOAT_EQ (expected, actual)
   EXPECT_DOUBLE_EQ (expected, actual)
   EXPECT_NEAR (expected, actual, absolute_range)
   ~~~

5. 死亡测试

   测试不通过，则直接退出
~~~
ASSERT_DEATH(statement, expected_message)
ASSERT_EXIT(statement, predicate, expected_message)
~~~


6. 使用gtest fixture做测试前准备和测试后处理

   ~~~C++
   class myTestFixture1: public ::testing::test {
        public:
        myTestFixture1( ) {
        // initialization code here
        }
        void SetUp( ) {
        // code here will execute just before the test ensues
        }
        void TearDown( ) {
        // code here will be called just after the test completes
        // ok to through exceptions from here if need be
        }
        ~myTestFixture1( ) {
        // cleanup any pending stuff, but no exceptions allowed
        }
        // put in any custom data members that you need
   };

   TEST_F (myTestFixture1, UnitTest1) {
    .
    }
    TEST_F (myTestFixture1, UnitTest2) {
    .
    }
   ​~~~
   ~~~

注意：这里每个TEST_F都会新生成一个myTestFixture1的对象。