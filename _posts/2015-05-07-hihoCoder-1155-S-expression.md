---
layout: default
title: hihoCoder 1155 S-expression（微软2016校园招聘在线笔试第二场 第四题）
comments: true
---

###1155 : S-expression[(原题链接)](http://hihocoder.com/problemset/problem/1155)
时间限制:5000ms
单点时限:1000ms
内存限制:256MB
###描述
S-expression is a prefix notation invented for and popularized by the programming language Lisp. Your task is to evaluate some simple S-expressions.

In this problem, S-expression is defined as:

1. An atom.
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. A nonnegative integer. Its value is the value of the integer.</br></br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. A boolean value, true or false. Its value is the boolean value of itself.</br></br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. A variable, consisting of no more than 10 lower case letters, excluding reserved words "if", "let", "true" and "false". Its value is the value bound to the variable. If the varible is not bound yet, produce an error "Unbound Identifier". (See below for details)
</br></br>
2. ( f x ... )
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a. one of the following 4 forms: ( + x y ) ( - x y ) ( * x y ) ( / x y ) in which x and y are both S-expressions.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To evaluate the S-expression, you need to first evaluate x then evaluate y.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If the value of x or y is not an integer, produce an error "Type Mismatch".
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If both x and y are integers, its value is the their sum(x+y)/difference(x-y)/product(x*y)/quotient(x/y). The division is the same as the integer division ("/" operator) in C/C++/Java/C#, truncated division. If the value of y is zero, produce an error: "Division By Zero".
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;b. ( if a b c ) in which a, b and c are S-expressions.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To evaluate the S-expression, you need to evaluate a at first.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If the value of a is not a boolean value, produce an error: "Type Mismatch".
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If the value of a is true, evaluate b and the S-expression's value is the value of b.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If the value of a is false, evaluate c and the S-expression's value is the value of c.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that b or c may not be evaluated during the calculation.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c. ( let ( x a ) b ) in which x is a variable consisting of no more than 10 lower case letters, excluding reserved words "if", "let", "true" and "false", a and b are S-expressions.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To evaluate the S-expression, you need to first evaluate a.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Then bind the value of a to the variable x and evaluate b. Binding means if variable x appears in b, its value is the value of a. The S-expression's value is the value of b.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that if x is bound to another value in b, the outer binding is ineffective. For example the value of ( let ( x 1 ) ( let ( x 2 ) x ) ) is 2.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d. one of the following 3 forms: ( < x y ) ( > x y ) ( = x y ) in which x and y are S-expressions.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To evaluate the S-expression, you need to first evaluate x then evaluate y.
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If the value of x or y is not an integer, produce an error "Type Mismatch".
</br>
</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If both x and y are integers, its value is a boolean value indicating whether x < y, x > y or x = y is true.
</br>
</br>


Given an S-expression, output its value. If an error occurs stop the evaluation and output the error.

###输入
The first line contains an integer T (1 <= T <= 5), the number of testcases.

The following T lines each contain an S-expression consisting of no more than 200 characters.

It is guaranteed input S-expressions do not have any syntax error. The variables, parentheses, integers, true and false are seperated by at least one space.

For 20% of the data there is no "let" in the S-expressions.

For another 20% of the data there is no "let", "if" in the S-expressions.

For another 20% of the data output contains no errors.

For 100% of the data the integers in the whole evaluation will not exceed 32bit signed integers.

###输出
For each testcase output the value of the S-expression or one of the three errors "Division By Zero", "Type Mismatch" and "Unbound Identifier" without quotes.

###样例输入

```
2
( + ( - 3 2 ) ( * 4 5 ) )
( let ( x 4 ) ( if true x y ) )
```
###样例输出

```
21
4
```

###Solution
本质上，这道题是要实现一个非常轻量级的解释器。

主要思路是，递归地处理输入流，边输入边解析出结果。主要逻辑并不难，只是限于本人水平，实现地比较繁琐。

观察：

1. 可以分成两类，一类是原子性的term，比如"true", "false", "10"等，可以直接解析结果（作为递归的Base情况）；另一类是非原子性的term，依赖于subterm，
比如"(if true 1 2)"， 整个term必须先解析完subterm，即"true", "1", "2"，而解析subterm可以递归处理；</br>
2. 区分原子性的term和非原子性的term：如果有括号包起来，按照语义显然是非原子性的term，否则是原子性的term；</br>
3. 按照evaluate的次序，如果存在错误，需要筛选出先发生的错误，过滤掉后发生的错误。特别注意if语句的短路evaluate，要根据bool值过滤掉不执行的语句。

代码如下：
***

```cpp
#include<iostream>
#include<map>
#include<string>
#include<sstream>
using namespace std;

class Result{
public:
    enum Type {Bool_Val = 0, Int_Val = 1, Division_By_Zero = 2,
               Type_Mismatch = 3, Unbound_Identifier = 4};
    Type t;
    int i;
    bool b;
};

inline bool isValid(Result &r){
    return (r.t == Result::Bool_Val || r.t == Result::Int_Val);
}

Result input(map<string, Result> &m){
    string s;
    Result r;
    cin >> s;
    if(s != "("){ // value or variable
        if(s == "true"){
            r.t = Result::Bool_Val;
            r.b = true;
        }else if(s == "false"){
            r.t = Result::Bool_Val;
            r.b = false;
        }else if(s[0] >= '0' && s[0] <= '9'){
            r.t = Result::Int_Val;
            stringstream ss(s);
            ss >> r.i;
        }else{ // variable name
            if(m.count(s)){
                r = m[s];
            }else{
                r.t = Result::Unbound_Identifier;
            }
        }
    }else{
        cin >> s;
        if(s == "+" || s == "-" || s == "*" || s == "/"){
            Result r1 = input(m);
            Result r2 = input(m);
            if(!isValid(r1)){
                r = r1;
            }else if(!isValid(r2)){
                r = r2;
            }else{
                if(r1.t != Result::Int_Val || r2.t != Result::Int_Val){
                    r.t = Result::Type_Mismatch;
                }else{
                    r.t = Result::Int_Val;
                    switch(s[0]){
                    case '+':
                        r.i = r1.i + r2.i;
                        break;
                    case '-':
                        r.i = r1.i - r2.i;
                        break;
                    case '*':
                        r.i = r1.i * r2.i;
                        break;
                    case '/':
                        if(r2.i == 0){
                            r.t = Result::Division_By_Zero;
                        }else{
                            r.i = r1.i / r2.i;
                        }
                        break;
                    }
                }
            }
        }else if(s == "<" || s == ">" || s == "="){
            Result r1 = input(m);
            Result r2 = input(m);
            if(!isValid(r1)){
                r = r1;
            }else if(!isValid(r2)){
                r = r2;
            }else{
                if(r1.t != Result::Int_Val || r2.t != Result::Int_Val){
                    r.t = Result::Type_Mismatch;
                }else{
                    r.t = Result::Bool_Val;
                    switch(s[0]){
                    case '<':
                        r.b = r1.i < r2.i;
                        break;
                    case '>':
                        r.b = r1.i > r2.i;
                        break;
                    case '=':
                        r.b = r1.i == r2.i;
                        break;
                    }
                }
            }
        }else if(s == "if"){
            Result r1 = input(m);
            Result r2 = input(m);
            Result r3 = input(m);
            if(!isValid(r1)){
                r = r1;
            }else{
                if(r1.t != Result::Bool_Val){
                    r.t = Result::Type_Mismatch;
                }else{
                    if(r1.b){ // true, then evaluate r2
                        r = r2;
                    }else{ // false, then evaluate r3
                        r = r3;
                    }
                }
            }
        }else if(s == "let"){
            bool existedInMap;
            Result tmp;
            string name;
            cin >> s; // "("
            cin >> name;
            existedInMap = m.count(name) > 0 ? true: false;
            if(existedInMap) tmp = m[name];
            Result r1 = input(m);
            cin >> s; // ")"
            m[name] = r1;
            Result r2 = input(m);
            m.erase(name);
            if(existedInMap) m[name] = tmp;
            if(!isValid(r1)){
                r = r1;
            }else if(!isValid(r2)){
                r = r2;
            }else{
                r = r2;
            }
        }else{
            cerr << "You can never go here!" << endl;
        }
        cin >> s; // ")"
    }

    return r;
}

int main(){
    int T;
    cin >> T;
    while(T--){
        map<string, Result> m;
        Result r = input(m);
        switch(r.t){
        case Result::Bool_Val:
            cout << boolalpha << r.b;
            break;
        case Result::Int_Val:
            cout << r.i;
            break;
        case Result::Division_By_Zero:
            cout << "Division By Zero";
            break;
        case Result::Type_Mismatch:
            cout << "Type Mismatch";
            break;
        case Result::Unbound_Identifier:
            cout << "Unbound Identifier";
            break;
        }
        cout << endl;
    }
    return 0;
}

```
