### 帧同步常见问题点

帧同步的原理是每个客户端和服务器在相同的初始条件下, 以同样的时机和顺序执行相同的战斗输入,得到彼此一致的战斗状态.

战斗不同步一旦产生,排查问题通常十分困难, 因此编写相关代码时需要我们特别注意.

下面总结了一些可能犯错误的地方.

#### 使用了浮点数

浮点数运算在不同架构的机器上不保证一致性, 因此战斗逻辑中的计算只能用定点数代替浮点数.

正确做法: **只使用定点数**

#### 容器顺序

* 无序容器的遍历

  考虑以下代码:

  ```c++
  std::unordered_map<int, FUnit*> Units;
  for(FUnit* U : Units)
  	DoSomething(U);   // 执行一些游戏逻辑
  ```

  该代码可能导致不同步. 原因是unordered_map可能在不同机器上以不同的顺序被遍历, 导致逻辑执行的先后顺序发生改变.

  正确做法: **慎重使用无序容器,相关逻辑不依赖遍历元素的顺序**

* 指针作为key

  考虑

  ```
  std::map<FUnit*, SomeDataStructure> UnitMap;
  for(auto& Pair : UnitMap)
  	DoSomething(Pair);
  ```

  该代码可能导致不同步, 原因是指针的值在不同机器上没有一致性保证, 遍历该有序容器的顺序也没有一致性保证.

  正确做法: **map类型的key应当使用稳定的数据,例如单位的id**

* std::sort

  当使用std::sort对容器进行排序时, 要特别注意是否会出现相等元素. 尤其是传入自定义比较器的情况下.

  考虑

  ```
  std::vector<FUnit*> SomeUnits;
  auto Comparer = [SomePosition](const FUnit* A, const FUnit* B)->bool
  {
  	return Distance(A->Position, SomePosition) < Distance(B->Position, SomePosition);
  }
  std::sort(SomeUnits.begin(), SomeUnits.end(), Comparer);
  // do something with SomeUnits..
  ```

  上面的代码将单位数组按照单位到某点距离进行排序. 该代码的排序结果不具备一致性保证. 原因是如果出现两单位到该点距离相等, 那么这两单位在排序完的结果数组中的顺序将不确定.

  解决办法: 

  * **修改比较器, 使其总是比较出确定的顺序关系**

    上面代码的比较器可以修改成

    ```
    auto Comparer = [SomePosition](const FUnit* A, const FUnit* B)->bool
    {
    	auto DistanceA = Distance(A->Position, SomePosition);
    	auto DistanceB = Distance(B->Positionm SomePosition);
    	
    	if(DistanceA != DistanceB)
    		return DistanceA < DistanceB;
    	
        return A->GetID() < B->GetID();
    }
    ```

    当距离相等时, 使用单位的唯一id将其进一步区分开(注意, 不要用他们指针的大小关系来做区分, 否则会导致类似前面指针做key的问题)

  * 有时需要比较的对象可能没有稳定的唯一id, 这时可以使用**std::stable_sort**代替std::sort.

     stable_sort保证,当两个元素在比较器中相等, 排序过后它们的相对顺序与排序前相同.

#### 静态和全局变量

战斗逻辑中应尽量避免使用静态和全局变量. 如果使用, 需要确保上一局战斗该变量的状态不会遗留, 每一局该变量都会正确重置.
目前战斗中只有表格和配置数据使用了静态变量. 它们一般情况下启动游戏以后就不再发生变化.
因此可以接受.

#### 外部逻辑误用

战斗外部系统误调用战斗系统中包含副作用的函数

例如, 外部某次调用使用了战斗中的随机数发生器, 将使得该场战斗的随机数无法再保持一致.

正确做法: **外围系统调用核心层函数应反复检查没有任何副作用,是只读取数据不做任何状态修改.**

#### c++函数和表达式求参顺序问题

在c++中,函数和表达式的参数会先完成求值, 再代入函数进行调用. 

对于含有多个参数的函数或表达式, 其参数求值顺序是不确定的. 

例如

```
	DoSomething(Foo1(), Foo2()); // 根据c++标准, Foo1()和Foo2()调用顺序没有任何保证
	A = Foo3() + Foo4();         // 同样地, Foo3()和Foo4()调用顺序不确定
```

当计算参数的调用包含副作用时, 这些地方将引入不一致性.

正确做法: **所有参数应在函数和表达式外部先求值完毕再代入.**


​	
​	