Tags: c++


最近在自娱自乐c++的时候发现，使用了STL的程序，
比以前纯C的要慢很多。
(不过以我肤浅的看法，哪怕有性能损失，我也愿意用STL，因为它极大的方便了代码的编写)

后来经过排查后发现一段多次调用的函数拖慢了整个程序。
那段代码的功能是给一个容器引用参数，如果容器内有相同的元素，
就返回true，否则false。

刚开始的想法受Python影响了。用了set。
因为在Python中，下面的代码很方便的检测一个list中是否有重复值

    ```python
    has_duplicate = lambda iterable: len(iterable) != len(set(iterable))
    ```

上面的代码会对 **迭代器失效**， 先别在意。


所以我在C++中就写下了类似的代码：

    ```c++
    bool has_duplicate(const vector<int> &values)
    {
        set<int> tmp(values.begin(), values.end());
        return (tmp.size() != values.size());
    }

    ```


每次都要去构建一遍set，真的很慢…………

后来改为用算法库中的 sort 和 unique 配合，一下将程序运行速度提升了很多。


## 测试代码

下面的测试代码，显示，用set最慢，用算法库最快。
代码写的很渣，淡淡的蛋疼

    ```c++
    #include <iostream>
    #include <vector>
    #include <set>
    #include <algorithm>
    #include <ctime>

    using namespace std;

    class Test
    {
        public:
            Test(const vector<int>&v):values(v){}
            bool test_with_direct(void);
            bool test_with_set(void);
            bool test_with_unique(void);

        private:
            vector<int> values;
    };

    bool Test::test_with_direct(void)
    {
        for(vector<int>::iterator it=values.begin(); it<values.end()-1; it++)
        {
            for(vector<int>::iterator _it=it+1; _it<values.end(); _it++)
            {
                if(*it == *_it) return true;
            }
        }
        return false;
    }

    bool Test::test_with_set(void)
    {
        set<int> tmp(values.begin(), values.end());
        return (tmp.size() != values.size());
    }

    bool Test::test_with_unique(void)
    {
        sort(values.begin(), values.end());
        vector<int>::iterator unique_iter = unique(values.begin(), values.end());
        return (unique_iter != values.end());
    }


    int main(int argc, char *argv[])
    {
        vector<int> input_duplicate{1, 23, 4, 2, 5, 2, 6, 4, 12, 3, 56, 33, 7, 90};
        vector<int> input_unique{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14};

        Test test_1(input_duplicate);
        Test test_2(input_unique);

        const int rounds = 10000 * 100;
        time_t start, end;

        cout << "test direct" << endl;
        start = time(NULL);
        for(int i=0; i<rounds; i++) test_1.test_with_direct();
        end = time(NULL);
        cout << "cost " << end-start << " seconds" << endl;


        cout << "test set" << endl;
        start = time(NULL);
        for(int i=0; i<rounds; i++) test_1.test_with_set();
        end = time(NULL);
        cout << "cost " << end-start << " seconds" << endl;

        cout << "test unique" << endl;
        start = time(NULL);
        for(int i=0; i<rounds; i++) test_1.test_with_unique();
        end = time(NULL);
        cout << "cost " << end-start << " seconds" << endl;

        return 0;
    }

    ```


## 结果

首先进行编译

    ```bash
    g++ test.cpp -o test -Wall -std=c++0x
    ```

**不要**用 -OX 去优化编译，这样就看不动效果了。运行./test便能看到结果:

*   test_with_set 最慢，test_with_unique 最快
*   如果用test_2 来测试，test_with_direct 最慢。因为得把所有元素全部历遍对此一遍


