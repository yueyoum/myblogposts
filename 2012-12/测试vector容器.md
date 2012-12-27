Tags: c++

最近在学习C++ STL容器和算法库，到目前有两点感受：
*   确实很方便。某些时候感觉就像写Python一样自然
*   一旦报错，满屏幕的错误提示……


在瞎试 vector 的时候发现一个问题。看下面代码：

    ```c++
    #include <iostream>
    #include <vector>

    using namespace std;

    class Test
    {
        public:
            Test(){cout << "Test init" << endl;}
            Test(const Test&){cout << "Test copy" << endl;}
            ~Test(){cout << "Test die" << endl;}
    }

    int main(void)
    {
        vector<Test> tests;
        Test t1;
        Test t2;
        Test t3;

        tests.push_back(t1);
        tests.push_back(t2);
        tests.push_back(t3);

        return 0;
    }
    ```


这是一个 Hello World 级别的代码。运行它，看看结果是什么？

    ```bash
    g++ test.cpp -o test -Wall
    ./test

    #output
    Test init
    Test init
    Test init
    Test copy
    Test copy
    Test copy
    Test die
    Test copy
    Test copy
    Test copy
    Test die
    Test die
    Test die
    Test die
    Test die
    Test die
    Test die
    Test die
    ```

为什么会有这么多的 copy 和 die ？

# 拷贝

vector<Test> tests; 这种申明，是要把 Test 对象拷贝入 tests 的.
做个实验，把Test的拷贝构造函数改成：

    ```c++
    Test(const Test&) = delete;
    ```

这里用到了C++11 的新特性，特殊函数 = delete; 会阻止编译器启动添加相应函数。
特殊函数指的就是构造函数，拷贝构造函数，析构函数。大家可以自己google。
因为是c++11的新特性，所以得这样编译：

    ```bash
    g++ test.cpp -o test -Wall -std=c++0x
    ```

当尝试编译的时候，便报错了，

    ```bash
    error: use of deleted function ‘Test::Test(const Test&)
    ```

很显然，因为要把 Test 放入 tests，是一个拷贝的过程，而 Test 又没有拷贝构造函数。自然编译不过。
然后我们将构造函数改回来.


# vector连续存放

vector容器有非常高的随机访问效率，就是因为vector像数组那样
连续存储。但vector能够动态增大。问题就在这里。当当前容量不够时，还有新元素要
push_back,vector为了保证连续，会申请新的，更大的空间，把以前的元素拷贝到新空间，
再加入新元素。所有就看到输出中有6个 Test copy.

同时也解释了为什么 vector 容器不适合做 insert, 而 list 容器就适合insert.
它们实现的方式不一样。

再来个实验： 在vector<Test> tests; 的下面加上:

    ```c++
    test.reserve(3);
    ```

reserve 可以设定容器的容量，我们直接设定为3. 再次编译看运行结果

    ```bash
    Test init
    Test init
    Test init
    Test copy
    Test copy
    Test copy
    Test die
    Test die
    Test die
    Test die
    Test die
    Test die
    ```

这次的结果就与构想的一样 。6次die试因为本身的3个 Test 实例, 和 tests 中的3个实例


# 正确的做法

上面只是演示vector的特性，所以方便，灵活的做法应该还是使用指针.
将 main函数 修改为：

    ```c++
    vector<Test*> tests;
    Test *t1 = new Test;
    Test *t2 = new Test;
    Test *t3 = new Test;

    tests.push_back(t1);
    tests.push_back(t2);
    tests.push_back(t3);

    for(auto &t: tests)
        delete t;

    return 0;
    ```

编译运行，就得到了漂亮的输出:

    ```bash
    g++ test.cpp -o test -Wall -std=c++0x

    #output:
    Test init
    Test init
    Test init
    Test die
    Test die
    Test die
    ```

注意：上面，还使用c++11中的新式for

