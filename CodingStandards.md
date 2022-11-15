#### 要求

* const 和 constexpr 

  编译期常量应使用constexpr, 语义更准确. const更准确的含义是只读

* NULL 和 nullptr

  指针类型的空值使用 nullptr

* 使用 auto 关键字省略迭代器类型时, 注意区分左右值和常量性.

  ```
  std::map<int, int> TestMap;
  auto It = TestMap::find(3);	//正确, 右边表达式返回右值
  auto& It = TestMap::find(3);	//不正确, 右值绑定到左值, 但msvc下依然可以编译通过
  auto&& It = TestMap::find(3);   //正确, 万能引用
  const auto& It = TestMap::find(3);	//正确, 右值可以绑定到常量引用
  ```

* 能使用 c++ static_cast 代替c语言强转的地方, 一律使用static_cast.

* 用于客户端服务器数据交换的基础数据类型, 应使用平台无关的定长类型. 例如 int 和 long 应该使用 int32_t来

  代替.

* 自定义读写的网络消息结构体, 不要用pack(1)宏包裹.
* 客户端和引擎相关的字面字符串都应该用TEXT宏包裹
* 如果析构和构造函数什么也不做, 可以使用=default 或者干脆不定义

#### 推荐

* 使用using代替typedef, using具备适配模板的能力
* 头文件include保护推荐使用 pragma once, 头文件改名时无需改动, 无需担心同名头文件





