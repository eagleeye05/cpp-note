# 30-包含min函数的栈

[OJ链接](https://www.nowcoder.com/practice/4c776177d2c04c2494f2555c9fcc1e49?tpId=13&tqId=11173&tPage=1&rp=4&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking)

**题目描述**

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的min函数。在该栈中，调用min、push及pop的时间复杂度都是O(1)。

**思路分析**

使用一个辅助栈来存储相关的最小值：

1. 当value小于当前最小值时，把value压入栈；
2. 否则，把当前最小值再次压入栈；

**具体思路**

```c++
class Solution {
public:
    void push(int value) {
        stack1.push(value);
        if(stack2.empty() || value < stack2.top()){
            stack2.push(value);
        }
        else{
            stack2.push(stack2.top());
        }
    }
    void pop() {
        if(stack1.empty()){
            throw new exception();
        }
        stack1.pop();
        stack2.pop();
    }
    int top() {
        if(stack1.empty()){
            throw new exception();
        }
        return stack1.top();
    }
    int min() {
        if(stack1.empty()){
            throw new exception();
        }
        return stack2.top();
    }
private:
    stack<int> stack1;
    stack<int> stack2;
};
```

空间复杂度：O(n)