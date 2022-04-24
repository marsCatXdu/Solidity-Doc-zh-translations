# Solidity by Example - 基础

> 原文：[Solidity by Example | 0.8.10 (solidity-by-example.org)](https://solidity-by-example.org/)
>
> [对应 commit]([solidity-by-example/solidity-by-example.github.io at c517d5c894bb08117d816b92b48615dd20e1608d](https://github.com/solidity-by-example/solidity-by-example.github.io/tree/c517d5c894bb08117d816b92b48615dd20e1608d))
>
> 某些地方增加了一些我觉得有必要加的内容

## Hello World

`pragma` 关键字用于指定 Solidity 编译器版本

```solidity
// SPDX-License-Identifier: MIT
// 该 pragma 要求编译器版本大于等于 0.8.10 小于 0.9.0
pragma solidity ^0.8.10;

contract HelloWorld {
    string public greet = "Hello World!";
}
```

- `Pragma` 关键字用于开关某些编译器特性或要求编译器对某些东西进行检查（比如这里要对编译器版本有要求）

## 第一个 App

这段代码能够实现一个整数值的加、减、取值，该整数存储在合约中

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Counter {
		/**
		 * 这样声明的变量是【状态变量（ State variable ）】
		 * 状态变量会在合约存储（ Contract storage ）中永久保存。
		 * public 修饰符意味着该变量是外部可读的（比如其他合约），但外部不可写
		 */
    uint public count;

		/**
		 * 函数的 public 修饰符说明该函数可以从合约内部和外部进行调用
		 */
    // 获取成员变量 count 的值
    function get() public view returns (uint) {
        return count;
    }

    // count 值 +1
    function inc() public {
        count += 1;
    }

    // count 值 -1
    function dec() public {
        // count 值为 0 时会执行失败
        count -= 1;
    }
}
```

- 函数的view 修饰符说明该函数不会修改【状态】。下面是一些会修改状态的行为：
  - 写状态变量；
  - 触发事件；
  - 创建其他合约；
  - 使用 `selfdestruct`；
  - 通过 call 发送 Ether ；
  - 调用没有被标记为 `view` 和 `pure` 的函数；
  - 使用低层调用（ low-level calls ）；
  - 使用包含某些操作码（ opcode ）的內联汇编



## 基本数据类型

本段介绍一些 Solidity 中的基本数据类型：

- boolean
- uint
- int
- address

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Primitives {
    bool public boo = true;

    /*
    uint 是无符号整数，uint 有多种大小可选
        uint8      8bit 长
        uint16     16bit 长
        ...
        uint256    256bit 长
    */
    uint8 public u8 = 1;
    uint public u256 = 456;
    uint public u = 123; // 只写 uint 代表 uint256

    /*
    int 是有符号整数
    同样，从 int8 也有多种大小可选
    */
    int8 public i8 = -1;
    int public i256 = 456;
    int public i = -123; // int 相当于 int256

    // 取整数的最小值和最大值
    int public minInt = type(int).min;
    int public maxInt = type(int).max;

    address public addr = 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;

    /*
    byte 类型代表一些字节组成的序列。
    Solidity 有两种字节数组类型：定长和可变长度字节数组
    “bytes” 代表一个可变长的字节数组，相当于 byte[]
    */
    bytes1 a = 0xb5; //  [10110101]
    bytes1 b = 0x56; //  [01010110]

    // 未赋值变量的默认值如下：
    bool public defaultBoo;  // false
    uint public defaultUint; // 0
    int public defaultInt;   // 0
    // address 类型是一个 20 字节长度的账户地址
    address public defaultAddr; // 0x0000000000000000000000000000000000000000
}
```



## 变量

Solidity 中的变量可以分为三类：

- local（局部变量）：
  - 在函数中声明
  - 不在区块链上存储
- state（状态变量）：
  - 在函数外声明
  - 在区块链上存储
- global（全局变量）：提供区块链相关的信息

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Variables {
    // 这样声明在函数外面的状态变量，会被存储在区块链上
    string public text = "Hello";
    uint public num = 123;

    function doSomething() public {
        // 函数内声明的局部变量不会存在区块链上
        uint i = 456;

        // 下面是一些全局变量
        uint timestamp = block.timestamp; // 当前区块的时间戳
        address sender = msg.sender; // 合约调用者的地址
    }
}
```



## 常量

常量是不能修改值的变量，它们会被硬编码在合约中，使用常量能减少 gas 消耗

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Constants {
    // 常量变量名通常全大写表示
    address public constant MY_ADDRESS = 0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc;
    uint public constant MY_UINT = 123;
}
```



## Immutable（不可变）

不可变变量类似于常量，其可以在构造函数中赋值，之后就不能再修改值了

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Immutable {
    // 不可变变量名也通常是全大写的，与常量相同
    address public immutable MY_ADDRESS;
    uint public immutable MY_UINT;

    constructor(uint _myUint) {
        MY_ADDRESS = msg.sender;
        MY_UINT = _myUint;
    }
}
```



## 读写状态变量

要写状态变量，需要通过发送交易（ transaction ）来进行。但读状态变量是不需要交易费的

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract SimpleStorage {
    // 存储整数的状态变量
    uint public num;

    // 写状态变量的函数要使用交易来触发
    function set(uint _num) public {
        num = _num;
    }

    // 读状态变量不需要发送交易
    function get() public view returns (uint) {
        return num;
    }
}
```



## Ether 和 Wei

交易需要使用 ether 来支付费用，1 ether = 10^18 wei

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract EtherUnits {
    uint public oneWei = 1 wei;
    // 1 wei 的值等于整数 1
    bool public isOneWei = 1 wei == 1;

    uint public oneEther = 1 ether;
    // 1 ether 等于 10^18 wei
    bool public isOneEther = 1 ether == 1e18;
}
```



## Gas

交易需要支付的 ether 值等于【 `gas spent` * `gas price` 】，其中 `gas` 是我们的代码使用 EVM 虚拟机进行计算的计算量单位，`gas spent` 是该交易中总共消耗掉的 gas 量，`gas price` 是你愿意为每个 gas 付的 ether 价。

gas 价格更高的交易会在一批交易的处理中有更高的优先级，会被更快地加入到区块中。没有用完的 gas 会被退款回来。

GasLimit：在一个交易中，能消耗的 gas 是有上限的。上限分两种：

- `gas limit`：调用者愿意支付的最大 gas 量，由交易的发出者设置；
- `block gas limit`：一个块允许的最大 gas 量，由网络确定

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Gas {
    uint public i = 0;

		// 用完所有 gas 会导致交易失败，所有状态改变都将会滚，用掉的 gas 也不会退款
    function forever() public {
    		// 这里用一个无限循环消耗掉所有的 gas
        while (true) {
            i += 1;
        }
    }
}
```



## if / else

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract IfElse {
    function foo(uint x) public pure returns (uint) {
        if (x < 10) {
            return 0;
        } else if (x < 20) {
            return 1;
        } else {
            return 2;
        }
    }

    function ternary(uint _x) public pure returns (uint) {
        // if else 的三元表达式写法
        return _x < 10 ? 1 : 2;
    }
}
```



## 循环

solidity 支持 for，while，do while 循环，用法与多数常见语言相同

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Loop {
    function loop() public {
        // for 循环
        for (uint i = 0; i < 10; i++) {
            if (i == 3) {
                // 使用 continue 来跳过循环
                continue;
            }
            if (i == 5) {
                // 使用 break 退出循环
                break;
            }
        }

        // while 循环
        uint j;
        while (j < 10) {
            j++;
        }
    }
}
```



## 映射（ Mapping ）

映射创建语法：`mapping(keyType => valueType)`

`keyType` 可以是任意的 solidity 内置值类型，如 bytes、string 或 contract。

`valueType` 可以是包括另一个映射和数组在内的任何数据类型。

注意：映射不可迭代

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Mapping {
    // 从 address 到 uint
    mapping(address => uint) public myMap;

    function get(address _addr) public view returns (uint) {
        // Mapping 总是有返回值的，即使没有值也会返回一个默认值
        return myMap[_addr];
    }

    function set(address _addr, uint _i) public {
        // 修改给定地址的映射值
        myMap[_addr] = _i;
    }

    function remove(address _addr) public {
        // 将给定地址的映射值重设为默认值
        delete myMap[_addr];
    }
}

contract NestedMapping {
    // 嵌套映射：将地址值映射到另一个映射
    mapping(address => mapping(uint => bool)) public nested;

    function get(address _addr1, uint _i) public view returns (bool) {
    		// 像下面这样从嵌套映射中取值。即使从没初始化过也是能取的
        return nested[_addr1][_i];
    }

    function set(
        address _addr1,
        uint _i,
        bool _boo
    ) public {
        nested[_addr1][_i] = _boo;
    }

    function remove(address _addr1, uint _i) public {
        delete nested[_addr1][_i];
    }
}
```



## 数组

Solidity 中的数组大小可以是固定的（编译时确定）也可以是动态的

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Array {
    // 几种初始化数组的方式
    uint[] public arr;
    uint[] public arr2 = [1, 2, 3];
    // 定长数组，所有元素的初始值为 0
    uint[10] public myFixedSizeArr;

    function get(uint i) public view returns (uint) {
        return arr[i];
    }

    // Solidity 可以直接返回整个数组。但应该避免这样取可以无限增长的变长数组
    function getArr() public view returns (uint[] memory) {
        return arr;
    }

    function push(uint i) public {
        // 向数组尾部加一个值，会使数组长度 +1
        arr.push(i);
    }

    function pop() public {
        // 移除末尾元素，会使数组长度 -1
        arr.pop();
    }

    function getLength() public view returns (uint) {
        return arr.length;
    }

    function remove(uint index) public {
        // delete 会将对应下标处的值改为默认值（0），但不会让数组长度 -1
        delete arr[index];
    }

    function examples() external {
        // 在 memory 中创建数组。memory 中只能创建定长数组
        uint[] memory a = new uint[](5);
    }
}
```

### 移除数组元素的例子

可以通过将元素从右向左移动来移除数组元素

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract ArrayRemoveByShifting {
    // [1, 2, 3] -- remove(1) --> [1, 3, 3] --> [1, 3]
    // [1, 2, 3, 4, 5, 6] -- remove(2) --> [1, 2, 4, 5, 6, 6] --> [1, 2, 4, 5, 6]
    // [1, 2, 3, 4, 5, 6] -- remove(0) --> [2, 3, 4, 5, 6, 6] --> [2, 3, 4, 5, 6]
    // [1] -- remove(0) --> [1] --> []

    uint[] public arr;

    function remove(uint _index) public {
        require(_index < arr.length, "index out of bound");

        for (uint i = _index; i < arr.length - 1; i++) {
            arr[i] = arr[i + 1];
        }
        arr.pop();
    }

    function test() external {
        arr = [1, 2, 3, 4, 5];
        remove(2);
        // [1, 2, 4, 5]
        assert(arr[0] == 1);
        assert(arr[1] == 2);
        assert(arr[2] == 4);
        assert(arr[3] == 5);
        assert(arr.length == 4);

        arr = [1];
        remove(0);
        // []
        assert(arr.length == 0);
    }
}
```

## 枚举（Enum）

Solidity 的枚举类型适合用来进行模型选择和跟踪 state 变化（原文：Solidity supports enumerables and they are useful to model choice and keep track of state.）。枚举类型可以在合约外定义。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Enum {
    // 用来表示运输状态的枚举
    enum Status {
        Pending,
        Shipped,
        Accepted,
        Rejected,
        Canceled
    }

		// 枚举类型的默认初始值是定义中的第一个类型，本例中是“Pending”
    Status public status;

    // 该函数返回一个 uint 值
    // Pending  - 0
    // Shipped  - 1
    // Accepted - 2
    // Rejected - 3
    // Canceled - 4
    function get() public view returns (Status) {
        return status;
    }

    // 通过在输入中传入 uint 来修改 status
    function set(Status _status) public {
        status = _status;
    }

    // 修改 status 值的方式如下
    function cancel() public {
        status = Status.Canceled;
    }

    // delete 算符会将枚举值重置为 0
    function reset() public {
        delete status;
    }
}
```

### 声明和 import 枚举

可以在一个单独的文件中声明枚举，然后在另一个文件中 import。

声明：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;
// This is saved 'EnumDeclaration.sol'

enum Status {
    Pending,
    Shipped,
    Accepted,
    Rejected,
    Canceled
}
```

import 上面枚举的文件：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

import "./EnumDeclaration.sol";

contract Enum {
    Status public status;
}
```

## 结构体（ struct ）

可以通过创建 `struct` 来定义我们自己的数据类型。

`struct` 可以在合约外进行声明，然后通过 import 来使用（和上面的枚举比较像）

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Todos {
    struct Todo {
        string text;
        bool completed;
    }

    // "Todo" 结构体的数组
    Todo[] public todos;

    function create(string memory _text) public {
        // 下面是三种初始化结构体的方法
        // - 像调用函数一样初始化
        todos.push(Todo(_text, false));

        // - 通过键值指定来初始化
        todos.push(Todo({text: _text, completed: false}));

        // - 初始化一个空结构体，然后修改值
        Todo memory todo;
        todo.text = _text;
        // 不赋值的话，todo.completed 会被默认初始化为 false

        todos.push(todo);
    }

    // solidity 会为 todos 数组创建一个默认的getter 函数，不是必须手动创建的
    function get(uint _index) public view returns (string memory text, bool completed) {
        Todo storage todo = todos[_index];
        return (todo.text, todo.completed);
    }

    // 修改 text
    function update(uint _index, string memory _text) public {
        Todo storage todo = todos[_index];
        todo.text = _text;
    }

    // 修改 completed
    function toggleCompleted(uint _index) public {
        Todo storage todo = todos[_index];
        todo.completed = !todo.completed;
    }
}
```

### 声明和 import 结构体

定义结构体的文件：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

struct Todo {
    string text;
    bool completed;
}
```

import 上面定义的结构体的文件

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

import "./StructDeclaration.sol";

contract Todos {
    // 'Todo' 结构体的数组
    Todo[] public todos;
}
```



## 数据存储位置：Storage, Memory 和 Calldata

声明变量时，可以通过 `storage`, `memory`, `calldata` 来显式指定希望将这个变量的数据存到哪里：

- `storage` - 变量是 state 变量，会被存储到区块链上；
- `memory` - 变量存在 memory 中，只在函数调用过程中存在；
- `calldata` - 用于存储函数参数的特殊位置

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract DataLocations {
    uint[] public arr;		// 直接摆在合约里面，不加修饰的是 state，存在 storage 中
    mapping(uint => address) map;
    struct MyStruct {
        uint foo;
    }
    mapping(uint => MyStruct) myStructs;

    function f() public {
        // 使用 state 变量调用 _f 函数
        _f(arr, map, myStructs[1]);

        // 从映射中取一个结构体，存到 storage 中
        MyStruct storage myStruct = myStructs[1];
        // 在 memory 中创建一个结构体
        MyStruct memory myMemStruct = MyStruct(0);
    }

    function _f(
        uint[] storage _arr,
        mapping(uint => address) storage _map,
        MyStruct storage _myStruct
    ) internal {
        // 可以在这里操作 storage 变量
    }

    // memory 变量可以直接返回
    function g(uint[] memory _arr) public returns (uint[] memory) {
        // 可以在这里操作 memory 数组变量
    }

    function h(uint[] calldata _arr) external {
        // 可以在这里操作 calldata 数组变量
    }
}
```



## 函数

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Function {
    // 函数可以有多个返回值
    function returnMany()
        public
        pure
        returns (
            uint,
            bool,
            uint
        )
    {
        return (1, true, 2);
    }

    // 函数的返回值可以有名字
    function named()
        public
        pure
        returns (
            uint x,
            bool b,
            uint y
        )
    {
        return (1, true, 2);
    }

    // 可以通过名字来指定返回值
    // 像下面这种写法，是可以返回掉显式的 return 语句的
    function assigned()
        public
        pure
        returns (
            uint x,
            bool b,
            uint y
        )
    {
        x = 1;
        b = true;
        y = 2;
    }

    // 当调用另一个会返回多值的函数时，要使用解构赋值的表达式
    function destructuringAssignments()
        public
        pure
        returns (
            uint,
            bool,
            uint,
            uint,
            uint
        )
    {
        (uint i, bool b, uint j) = returnMany();

        // 不想要的值可以省略
        (uint x, , uint y) = (4, 5, 6);

        return (i, b, j, x, y);
    }

    // 映射不能作为函数的参数，也不能作为函数的返回值

    // 数组可以作为函数的参数
    function arrayInput(uint[] memory _arr) public {}

    // 数组也可以作为函数的返回值
    uint[] public arr;

    function arrayOutput() public view returns (uint[] memory) {
        return arr;
    }
}
```



## View 函数和 Pure 函数

getter 函数可以被声明为 view 或 pure

`view` 说明被其修饰的函数不会修改 state

`pure` 说明被其修饰的函数不会修改，也不会读取任何 state 变量

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract ViewAndPure {
    uint public x = 1;

    // view 函数不会修改 state
    function addToX(uint y) public view returns (uint) {
        return x + y;
    }

    // pure 函数既不会修改 state ，也不会读 state
    function add(uint i, uint j) public pure returns (uint) {
        return i + j;
    }
}
```



## 错误（ Error ）

如果一个交易中出现了错误，那么所有的 state 变动都会被回滚。可以使用 `require`, `revert` 和 `assert` 来抛出错误。

- `require` 用于在执行前验证输入和条件；
- `revert` 与 `require` 类似，可以看下面的代码了解相关细节；
- `assert` 用于检查永远不能为 false 的代码。如果出现断言失败（ failing assertion ）的情况，那很可能意味着相关的代码有 bug

使用自定的错误能节约 gas

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Error {
    function testRequire(uint _i) public pure {
        // require 可用于验证条件，例如：
        // - 输入
        // - 执行前的条件检查
        // - 调用其他函数的返回值
        require(_i > 10, "Input must be greater than 10");
    }

    function testRevert(uint _i) public pure {
        // revert 可以插入到正常逻辑中，可用于检查条件较复杂的情况
        // 下面代码的效果和上面的相同
        if (_i <= 10) {
            revert("Input must be greater than 10");
        }
    }

    uint public num;

    function testAssert() public view {
    		// assert 应该只被用于测试内部错误和检查不变的量

				// 下面检查 num 是否总等于 0
        assert(num == 0);
    }

    // 自定义 error
    error InsufficientBalance(uint balance, uint withdrawAmount);

    function testCustomError(uint _withdrawAmount) public view {
        uint bal = address(this).balance;
        if (bal < _withdrawAmount) {
            revert InsufficientBalance({balance: bal, withdrawAmount: _withdrawAmount});
        }
    }
}
```



## 函数修饰符（Function Modifier）

修饰符是可以在函数执行前和（或）执行后运行的代码。其可以被用于：

- 限制访问权限
- 验证输入
- 预防重入攻击

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract FunctionModifier {
    // 本例将用下面这些变量来演示修饰符的用法
    address public owner;
    uint public x = 10;
    bool public locked;

    constructor() {
        // 将交易发送方设置为该合约的所有者
        owner = msg.sender;
    }

    // 检查合约调用者是否为合约的所有者
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        // 下划线是只能在函数修饰符内使用的特殊字符，其用于告知 solidity
        // 继续执行剩下的代码
        _;
    }

    // 修饰符可以获取输入参数，这里用于检查并确保输入的地址不是 0 地址
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Not valid address");
        _;
    }

    function changeOwner(address _newOwner) public onlyOwner validAddress(_newOwner) {
        owner = _newOwner;
    }

    // 修饰符可以在函数执行之前或（和）之后调用。下面的修饰符用于防止函数
    // 在执行过程中被调用
    modifier noReentrancy() {
        require(!locked, "No reentrancy");

        locked = true;
        _;
        locked = false;
    }

    function decrement(uint i) public noReentrancy {
        x -= i;

        if (i > 1) {
            decrement(i - 1);
        }
    }
}
```



## 事件（Events）

`event` 允许合约向 ethereum 区块链写入日志，主要有下面两个用途：

- 监听事件并更新用户界面；
- 更廉价地存储数据

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Event {
    // 声明事件，声明时最多可以按顺序加三个参数。可以根据这三个参数过滤日志
    event Log(address indexed sender, string message);
    event AnotherLog();

    function test() public {
        emit Log(msg.sender, "Hello World!");
        emit Log(msg.sender, "Hello EVM!");
        emit AnotherLog();
    }
}
```



## 构造函数（ Constructor ）

构造函数是在合约创建时运行的函数，可以省略

使用带参数构造函数的例子：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// 父合约 X （下面的代码会继承这个合约）
contract X {
    string public name;

    constructor(string memory _name) {
        name = _name;
    }
}

// 父合约 Y
contract Y {
    string public text;

    constructor(string memory _text) {
        text = _text;
    }
}

// 下面展示了两种初始化带参数父合约的方式

// 在继承列表中传入参数
contract B is X("Input to X"), Y("Input to Y") {

}

contract C is X, Y {
    // 在构造函数中传递参数，写起来和函数修饰符类似
    constructor(string memory _name, string memory _text) X(_name) Y(_text) {}
}

// 父构造函数的调用顺序是由继承顺序决定的，与父合约构造函数在子构造函数列表中的排列顺序无关
// 见下面的例子

// 构造函数调用顺序:
// 1. X
// 2. Y
// 3. D
contract D is X, Y {
    constructor() X("X was called") Y("Y was called") {}
}

// 构造函数调用顺序:
// 1. X
// 2. Y
// 3. E
contract E is X, Y {
    constructor() Y("Y was called") X("X was called") {}
}
```



## 继承

Solidity 支持多继承。合约可以使用 `is` 关键字来继承其他合约。

父合约中要由子合约重写的函数要用 `virtual` 关键字声明，子合约要重写父合约的函数要用 `override` 关键字声明。

父合约在子合约的继承列表中的顺序，必须是从最基础向派生的方向来排列的

> 原文：You have to list the parent contracts in the order from “most base-like” to “most derived”.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

/* 本代码示例中的继承关系图：
    A
   / \
  B   C
 / \ /
F  D,E

*/

contract A {
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

// 使用 'is' 关键字表示继承
contract B is A {
    // 子合约重写 A.foo()
    function foo() public pure virtual override returns (string memory) {
        return "B";
    }
}

contract C is A {
    // 子合约重写 A.foo()
    function foo() public pure virtual override returns (string memory) {
        return "C";
    }
}

// 一个合约可以继承多个父合约，当调用【在多个父合约中重复定义过】的函数时，
// 会在多个父合约中按继承顺序从右到左搜索，按深度优先规则来选择要执行的函数

contract D is B, C {
    // D.foo() 会返回 "C"
    // 因为 C 是定义了 foo() 函数的合约中，排在继承列表中最靠右边的
    function foo() public pure override(B, C) returns (string memory) {
        return super.foo();
    }
}

contract E is C, B {
    // E.foo() 会返回 "B"
    // 因为 B 是定义了 foo() 函数的合约中，排在继承列表中最靠右边的
    function foo() public pure override(C, B) returns (string memory) {
        return super.foo();
    }
}

// 父合约在子合约的继承列表中的顺序，必须是从最基础向派生的方向来排列的
// 调换 A、B 的顺序会引起编译错误
contract F is A, B {
    function foo() public pure override(A, B) returns (string memory) {
        return super.foo();
    }
}
```



## 屏蔽继承的状态变量（shadowing inherited state varibales）

与函数不同，状态变量是无法通过在子合约中重新声明的方式来覆盖的。

下例演示如何正确覆盖继承来的状态变量

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract A {
    string public name = "Contract A";

    function getName() public view returns (string memory) {
        return name;
    }
}

// Shadowing 是在 Solidity 0.6 版本后才支持的特性
// 下面这种写法是无法编译的
// contract B is A {
//     string public name = "Contract B";
// }

contract C is A {
    // 在构造函数中使用重新赋值的方法来覆盖状态变量：
    constructor() {
        name = "Contract C";
    }

    // C.getName 会返回 "Contract C"
}
```



## 调用父合约

子合约可以直接调用父合约的函数，也可以使用 `super` 关键字调用

当使用 `super` 进行调用时，所有继承的父合约函数都会被调用

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

/* 本代码示例中的继承关系图：
   A
 /  \
B   C
 \ /
  D
*/

contract A {
    // 下面是一个事件。可以在函数中调用事件，把事件记录到交易日志中
    // 本例使用事件追踪函数调用
    event Log(string message);

    function foo() public virtual {
        emit Log("A.foo called");
    }

    function bar() public virtual {
        emit Log("A.bar called");
    }
}

contract B is A {
    function foo() public virtual override {
        emit Log("B.foo called");
        A.foo();
    }

    function bar() public virtual override {
        emit Log("B.bar called");
        super.bar();
    }
}

contract C is A {
    function foo() public virtual override {
        emit Log("C.foo called");
        A.foo();
    }

    function bar() public virtual override {
        emit Log("C.bar called");
        super.bar();
    }
}

contract D is B, C {
    // 这里建议打开 remix ，动手试一试。
    // 调用 D.foo 时，尽管 D 继承了 A 、 B 和 C ，但实际上只先调用了 C 然后调了 A
    function foo() public override(B, C) {
        super.foo();
    }
		// 调用 D.bar 时，D 调了 C，然后调了 B ß，最后调了 A
		// 尽管用了两次 super （分别由 B 、 C 用过），但只调了一次 A 
    function bar() public override(B, C) {
        super.bar();
    }
}
```



## 可见性

函数可声明为以下几种类型：

- `public` - 任何合约及账户都可以调用
- `private` - 只有定义函数的合约能调用
- `internal` - 只有继承了 `internal` 函数的合约中
- `external` - 只有其他合约和账户能调用

状态变量可以声明为 `public`, `private` 或 `external`，但不能为 `external`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Base {
    // Private 函数只有定义函数的合约能调用
    // 继承该合约的子合约无法调用
    function privateFunc() private pure returns (string memory) {
        return "private function called";
    }

    function testPrivateFunc() public pure returns (string memory) {
        return privateFunc();
    }

    // Internal 函数
    // - 可在当前合约内调用
    // - 继承该合约的子合约内也可调用
    function internalFunc() internal pure returns (string memory) {
        return "internal function called";
    }

    function testInternalFunc() public pure virtual returns (string memory) {
        return internalFunc();
    }

    // Public 函数
    // - 当前合约中可调用
    // - 继承该合约的子合约内可调用
    // - 其他合约和账户可调用
    function publicFunc() public pure returns (string memory) {
        return "public function called";
    }

    // External 函数只能被其他合约和账户调用
    function externalFunc() external pure returns (string memory) {
        return "external function called";
    }

    // 下面的函数由于试图调用一个 external 函数，因此无法通过编译
    // function testExternalFunc() public pure returns (string memory) {
    //     return externalFunc();
    // }

    // 不同可见性的状态变量
    string private privateVar = "my private variable";
    string internal internalVar = "my internal variable";
    string public publicVar = "my public variable";
    // 状态变量不能是 external 的，因此下面的声明无法通过编译
    // string external externalVar = "my external variable";
}

contract Child is Base {
    // 派生合约是无法访问 private 函数和状态变量的
    // function testPrivateFunc() public pure returns (string memory) {
    //     return privateFunc();
    // }

    // Internal 函数可以在子合约中调用
    function testInternalFunc() public pure override returns (string memory) {
        return internalFunc();
    }
}
```



## 接口（ interface ）

可以通过声明 `interface` 的方式来与其他合约进行交互

接口：

- 不能包含任何函数的实现；
- 可以从其他接口派生；
- 所有函数必须是 external 的；
- 不能声明构造函数；
- 不能声明状态变量

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Counter {
    uint public count;

    function increment() external {
        count += 1;
    }
}

interface ICounter {
    function count() external view returns (uint);

    function increment() external;
}

contract MyContract {
    function incrementCounter(address _counter) external {
        ICounter(_counter).increment();
    }

    function getCount(address _counter) external view returns (uint) {
        return ICounter(_counter).count();
    }
}

// Uniswap 示例
interface UniswapV2Factory {
    function getPair(address tokenA, address tokenB)
        external
        view
        returns (address pair);
}

interface UniswapV2Pair {
    function getReserves()
        external
        view
        returns (
            uint112 reserve0,
            uint112 reserve1,
            uint32 blockTimestampLast
        );
}

contract UniswapExample {
    address private factory = 0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f;
    address private dai = 0x6B175474E89094C44Da98b954EedeAC495271d0F;
    address private weth = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    function getTokenReserves() external view returns (uint, uint) {
        address pair = UniswapV2Factory(factory).getPair(dai, weth);
        (uint reserve0, uint reserve1, ) = UniswapV2Pair(pair).getReserves();
        return (reserve0, reserve1);
    }
}
```



## Payable

被声明为 `payable` 的函数和地址可以在合约中接收 ether

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Payable {
    // Payable address 可接收 Ether
    address payable public owner;

    // Payable 构造函数可接收 Ether
    constructor() payable {
        owner = payable(msg.sender);
    }

		// 将 ether 存入本合约的函数。带 ether 调用该函数，则合约的余额会自动更新
    function deposit() public payable {}

    // 带 ether 调用非 payable 函数时会抛出错误
    function notPayable() public {}

    // 该函数能退还合约中所有 ether
    function withdraw() public {
        // 获取存储在该合约中的 ether 数量
        uint amount = address(this).balance;

        // 将所有 ether 发给所有者
        // 因为 owner 是 payable 的，所以能接收 ether
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Failed to send Ether");
    }

    // 从该合约发送 ether 到参数地址的函数
    function transfer(address payable _to, uint _amount) public {
        // 需要注意 "to" 要被声明为 payable
        (bool success, ) = _to.call{value: _amount}("");
        require(success, "Failed to send Ether");
    }
}
```



## 发送 Ether（transfer, send, call ）

**如何发送 ether？**

可以通过下面三个方式来向其他合约发送 ether：

- `transfer` ： 消耗 2300 gas，错误时抛出错误
- `send` ： 消耗 2300 gas，返回一个 bool 值
- `call` ： 转发所有的 gas 或设置 gas ，返回 bool 值

**如何接收 ether？**

能接收 ether 的合约必须包含至少一个下面的函数

- `receive() external payable`
- `fallback() external payable`

交易的 `msg.data` 为空时会调用 `receive()` ，否则会调用 `fallback()`

**应该用哪个方法？**

推荐使用与防重入手段相结合的 `call` 函数 （`call` in combination with re-entrancy guard）

通过下面的方法来防止重入：

- 在调用其他合约前完成所有的状态修改；
- 使用重入守卫修饰符（ re-entrancy guard modifier ）

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract ReceiveEther {
    /*
    如何判断调用 fallback() 还是 receive()：

           send Ether
               |
         msg.data is empty?
              / \
            yes  no
            /     \
receive() exists?  fallback()
         /   \
        yes   no
        /      \
    receive()   fallback()
    */

    // 接收 ether 的函数. msg.data 必须为空
    receive() external payable {}

    // 当 msg.data 非空时，调用 fallback()
    fallback() external payable {}

    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract SendEther {
    function sendViaTransfer(address payable _to) public payable {
        // 不推荐使用该函数发送 ether
        _to.transfer(msg.value);
    }

    function sendViaSend(address payable _to) public payable {
        // send 会返回表示发送结果的 bool 值。不推荐使用该函数。
        bool sent = _to.send(msg.value);
        require(sent, "Failed to send Ether");
    }

    function sendViaCall(address payable _to) public payable {
        // call 会返回表示发送结果的 bool 值。该例是推荐的用法
        (bool sent, bytes memory data) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}
```



## Fallback

`fallback` 是个不接收任何参数、没有任何返回值的函数，在下面两种场景下会触发执行：

- 调用了一个不存在的函数
- 直接向一个合约发送 ether ，但合约中没有 `receive()` 函数或 `msg.data` 非空

当被 `transfer` 或 `send` 调用时， `fallback` 有 2300 的 gas limit

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Fallback {
    event Log(uint gas);

    // Fallback 函数必须时 external 的.
    fallback() external payable {
        // send / transfer (发送 2300 gas 到这个 fallback 函数)
        // call (发送所有 gas)
        emit Log(gasleft());
    }

    // 检查该合约余额的工具函数
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract SendToFallback {
    function transferToFallback(address payable _to) public payable {
        _to.transfer(msg.value);
    }

    function callFallback(address payable _to) public payable {
        (bool sent, ) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}
```



## Call

`call` 是一个用于和其他合约交互的低层函数

当使用 `fallback` 发送 ether 时，推荐使用该函数

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Receiver {
    event Received(address caller, uint amount, string message);

    fallback() external payable {
        emit Received(msg.sender, msg.value, "Fallback was called");
    }

    function foo(string memory _message, uint _x) public payable returns (uint) {
        emit Received(msg.sender, msg.value, _message);

        return _x + 1;
    }
}

contract Caller {
    event Response(bool success, bytes data);

    // 假设合约 B 不知道合约 A 的代码，但知道合约 A 的地址和要调用的函数
    function testCallFoo(address payable _addr) public payable {
        // 可以发送 ether 并指定自定义的 gas 数量
        (bool success, bytes memory data) = _addr.call{value: msg.value, gas: 5000}(
            abi.encodeWithSignature("foo(string,uint256)", "call foo", 123)
        );

        emit Response(success, data);
    }

    // 调用不存在的函数，触发 fallback
    function testCallDoesNotExist(address _addr) public {
        (bool success, bytes memory data) = _addr.call(
            abi.encodeWithSignature("doesNotExist()")
        );

        emit Response(success, data);
    }
}
```



## Delegatecall

`delegatecall` 是类似于 `call` 的低层函数。当合约 `A` 对 `B` 执行 `delegatecal` 时，会使用合约 `A` 的 storage、`msg.sender` 和 `msg.value` 来执行 `B` 的代码

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// 注意：首先部署这个合约 B
contract B {
    // 注意: 本合约中 storage 的布局（layout）必须与 A 的相同
    uint public num;
    address public sender;
    uint public value;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
        value = msg.value;
    }
}

contract A {
    uint public num;
    address public sender;
    uint public value;

    function setVars(address _contract, uint _num) public payable {
        // A 的 storage 完成设置后, B 并没有被修改
        (bool success, bytes memory data) = _contract.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
}
```



## 函数选择器（ Function selector ）

当调用函数时，`calldata` 的前 4 bytes 被用于指定要调用的函数——这 4 字节就被称为 “函数选择器”。

例如下面的代码，使用 `call` 来调用 `addr` 地址合约中的 `transfer` 函数

```solidity
addr.call(abi.encodeWithSignature("transfer(address,uint256)", 0xSomeAddress, 123))
```

从 `abi.encodeWithSignature(...)` 函数返回的前 4 字节就是函数选择器。

预先计算并在代码中预置函数选择器，或许能够节省一丁点的 gas 开销。下面的例子中展示了如函数选择器是如何计算出来的

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract FunctionSelector {
    /*
    "transfer(address,uint256)"
    0xa9059cbb
    "transferFrom(address,address,uint256)"
    0x23b872dd
    */
    function getSelector(string calldata _func) external pure returns (bytes4) {
        return bytes4(keccak256(bytes(_func)));
    }
}
```



## 调用其他合约

一个合约可以使用两种方式来调用其他的合约

最简单的方式就是直接调用，如 `A.foo(x, y, z)`。另一个方式是使用低层的 `call` 函数。不是很推荐使用后者。

代码示例如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Callee {
    uint public x;
    uint public value;

    function setX(uint _x) public returns (uint) {
        x = _x;
        return x;
    }

    function setXandSendEther(uint _x) public payable returns (uint, uint) {
        x = _x;
        value = msg.value;

        return (x, value);
    }
}

contract Caller {
    function setX(Callee _callee, uint _x) public {
        uint x = _callee.setX(_x);
    }

    function setXFromAddress(address _addr, uint _x) public {
        Callee callee = Callee(_addr);
        callee.setX(_x);
    }

    function setXandSendEther(Callee _callee, uint _x) public payable {
        (uint x, uint value) = _callee.setXandSendEther{value: msg.value}(_x);
    }
}
```



## 在一个合约中创建新的合约

一个合约可以使用 `new` 关键字创建新的合约。从 0.8.0 开始，`new` 关键字可以通过指定 `salt` 选项来支持 `create2` 特性

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract Car {
    address public owner;
    string public model;
    address public carAddr;

    constructor(address _owner, string memory _model) payable {
        owner = _owner;
        model = _model;
        carAddr = address(this);
    }
}

contract CarFactory {
    Car[] public cars;

    function create(address _owner, string memory _model) public {
        Car car = new Car(_owner, _model);
        cars.push(car);
    }

    function createAndSendEther(address _owner, string memory _model) public payable {
        Car car = (new Car){value: msg.value}(_owner, _model);
        cars.push(car);
    }

    function create2(
        address _owner,
        string memory _model,
        bytes32 _salt
    ) public {
        Car car = (new Car){salt: _salt}(_owner, _model);
        cars.push(car);
    }

    function create2AndSendEther(
        address _owner,
        string memory _model,
        bytes32 _salt
    ) public payable {
        Car car = (new Car){value: msg.value, salt: _salt}(_owner, _model);
        cars.push(car);
    }

    function getCar(uint _index)
        public
        view
        returns (
            address owner,
            string memory model,
            address carAddr,
            uint balance
        )
    {
        Car car = cars[_index];

        return (car.owner(), car.model(), car.carAddr(), address(car).balance);
    }
}
```



## Try / Catch

`try / catch` 只能用于捕捉来自外部函数调用和合约创建的错误

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// 用于演示 try / catch 的外部合约
contract Foo {
    address public owner;

    constructor(address _owner) {
        require(_owner != address(0), "invalid address");
        assert(_owner != 0x0000000000000000000000000000000000000001);
        owner = _owner;
    }

    function myFunc(uint x) public pure returns (string memory) {
        require(x != 0, "require failed");
        return "my func was called";
    }
}

contract Bar {
    event Log(string message);
    event LogBytes(bytes data);

    Foo public foo;

    constructor() {
        // 这里的 foo 合约用于演示外部调用的 try catch
        foo = new Foo(msg.sender);
    }

    // 外部调用中的 try / catch 例子
    // tryCatchExternalCall(0) => Log("external call failed")
    // tryCatchExternalCall(1) => Log("my func was called")
    function tryCatchExternalCall(uint _i) public {
        try foo.myFunc(_i) returns (string memory result) {
            emit Log(result);
        } catch {
            emit Log("external call failed");
        }
    }

    // 合约创建中的 try / catch 例子
    // tryCatchNewContract(0x0000000000000000000000000000000000000000) => Log("invalid address")
    // tryCatchNewContract(0x0000000000000000000000000000000000000001) => LogBytes("")
    // tryCatchNewContract(0x0000000000000000000000000000000000000002) => Log("Foo created")
    function tryCatchNewContract(address _owner) public {
        try new Foo(_owner) returns (Foo foo) {
            // 这里可以使用 foo 变量
            emit Log("Foo created");
        } catch Error(string memory reason) {
            // 捕捉失败的 revert() 和 require()
            emit Log(reason);
        } catch (bytes memory reason) {
            // 捕捉失败的 assert()
            emit LogBytes(reason);
        }
    }
}
```



## 导入 （ import ）

Solidity 中可以导入本地或外部的文件

### 导入本地文件

本地文件结构：

```
├── Import.sol
└── Foo.sol
```

Foo.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

struct Point {
    uint x;
    uint y;
}

error Unauthorized(address caller);

function add(uint x, uint y) pure returns (uint) {
    return x + y;
}

contract Foo {
    string public name = "Foo";
}
```

Import.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// 从当前目录导入 Foo.sol
import "./Foo.sol";

// import {symbol1 as alias, symbol2} from "filename";
import {Unauthorized, add as func, Point} from "./Foo.sol";

contract Import {
    // 初始化 Foo.sol
    Foo public foo = new Foo();

    // 通过获取名称来测试 Foo.sol.
    function getFooName() public view returns (string memory) {
        return foo.name();
    }
}
```

### 从外部导入

可以直接用 github 网址 url 来导入文件

```solidity
// https://github.com/owner/repo/blob/branch/path/to/Contract.sol
import "https://github.com/owner/repo/blob/branch/path/to/Contract.sol";

// Example import ECDSA.sol from openzeppelin-contract repo, release-v4.5 branch
// https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";
```



## 库（ Library ）

库和合约类似，但库中不能声明状态变量，也不能发送 ether 。

如果库中的所有函数都是 internal 的，那么库会被嵌入到合约中。否则库必须被单独部署并在部署合约之前进行链接

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

library SafeMath {
    function add(uint x, uint y) internal pure returns (uint) {
        uint z = x + y;
        require(z >= x, "uint overflow");

        return z;
    }
}

library Math {
    function sqrt(uint y) internal pure returns (uint z) {
        if (y > 3) {
            z = y;
            uint x = y / 2 + 1;
            while (x < z) {
                z = x;
                x = (y / x + x) / 2;
            }
        } else if (y != 0) {
            z = 1;
        }
        // else z = 0 (default value)
    }
}

contract TestSafeMath {
    using SafeMath for uint;

    uint public MAX_UINT = 2**256 - 1;

    function testAdd(uint x, uint y) public pure returns (uint) {
        return x.add(y);
    }

    function testSquareRoot(uint x) public pure returns (uint) {
        return Math.sqrt(x);
    }
}

// 用于删除指定下标处的数组元素，并重新组织数组，防止出现空位
library Array {
    function remove(uint[] storage arr, uint index) public {
        // 将最后一个元素移动到删除导致的空位中
        require(arr.length > 0, "Can't remove from empty array");
        arr[index] = arr[arr.length - 1];
        arr.pop();
    }
}

contract TestArray {
    using Array for uint[];

    uint[] public arr;

    function testArrayRemove() public {
        for (uint i = 0; i < 3; i++) {
            arr.push(i);
        }

        arr.remove(1);

        assert(arr.length == 2);
        assert(arr[0] == 0);
        assert(arr[1] == 2);
    }
}
```



## ABI 编解码（ ABI Decode ）

`abi.encode` 能将数据编码为 `bytes`

`abi.decode` 能将 `bytes` 解码为数据

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract AbiDecode {
    struct MyStruct {
        string name;
        uint[2] nums;
    }

    function encode(
        uint x,
        address addr,
        uint[] calldata arr,
        MyStruct calldata myStruct
    ) external pure returns (bytes memory) {
        return abi.encode(x, addr, arr, myStruct);
    }

    function decode(bytes calldata data)
        external
        pure
        returns (
            uint x,
            address addr,
            uint[] memory arr,
            MyStruct memory myStruct
        )
    {
        // (uint x, address addr, uint[] memory arr, MyStruct myStruct) = ...
        (x, addr, arr, myStruct) = abi.decode(data, (uint, address, uint[], MyStruct));
    }
}
```



## 使用 Keccak256 计算 Hash

`keccak256` 可用于计算输入的 Keccak-256 hash 值

使用场景如：

- 根据输入来创建确定而唯一的 ID 值
- 提交-撤销方案（ Commit-Reveal scheme ）
- 进行紧凑密码学签名（使用 hash 签名替代更大的输入）

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

contract HashFunction {
    function hash(
        string memory _text,
        uint _num,
        address _addr
    ) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(_text, _num, _addr));
    }

    // hash 碰撞的例子
    // 当给 abi.encodePacked 传入超过一个动态数据类型的时候可能会出现 hash 碰撞的问题。
    // 在这种情况下应该使用 abi.encode 替代之
    function collision(string memory _text, string memory _anotherText)
        public
        pure
        returns (bytes32)
    {
        // encodePacked(AAA, BBB) -> AAABBB
        // encodePacked(AA, ABBB) -> AAABBB
        return keccak256(abi.encodePacked(_text, _anotherText));
    }
}

contract GuessTheMagicWord {
    bytes32 public answer =
        0x60298f78cc0b47170ba79c10aa3851d7648bd96f2f8e46a19dbc777c36fb0c00;

    // 这里的魔术字（ magic word ）是 "Solidity"
    function guess(string memory _word) public view returns (bool) {
        return keccak256(abi.encodePacked(_word)) == answer;
    }
}
```



## 验证签名

可以在链下对消息（ Messages ）进行签名，在链上使用智能合约验证

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

/* 签名验证

如何进行签名和验证
# 签名
1. 创建一个要签名的消息（ message ）
2. 对消息求 hash 值
3. 对 hash 进行签名 (在链下进行，确保私钥的安全性)

# 验证
1. 使用原信息重新产生 hash 值
2. 从签名和 hash 值恢复出签名者
3. 将恢复出来的 signer 和自称的 signer 进行比对
*/

contract VerifySignature {
    /* 1. 解锁 MetaMask 账户
    ethereum.enable()
    */

    /* 2. 获取要签名的消息 hash
    getMessageHash(
        0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C,
        123,
        "coffee and donuts",
        1
    )

    hash = "0xcf36ac4f97dc10d91fc2cbb20d718e94a8cbfe0f82eaedc6a4aa38946fb797cd"
    */
    function getMessageHash(
        address _to,
        uint _amount,
        string memory _message,
        uint _nonce
    ) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(_to, _amount, _message, _nonce));
    }

    /* 3. 对消息 hash 进行签名
    # 使用浏览器进行
    account = "copy paste account of signer here"
    ethereum.request({ method: "personal_sign", params: [account, hash]}).then(console.log)

    # 使用 web3 进行
    web3.personal.sign(hash, web3.eth.defaultAccount, console.log)

    不同账号会产生不同的签名
    0x993dab3dd91f5c6dc28e17439be475478f5635c92a56e17e82349d3fb2f166196f466c0b4e0c146f285204f0dcb13e5ae67bc33f4b888ec32dfe0a063e8f3f781b
    */
    function getEthSignedMessageHash(bytes32 _messageHash)
        public
        pure
        returns (bytes32)
    {
        /*
        签名是用下面的格式对 keccak256 hash 进行签名产生的：
        "\x19Ethereum Signed Message\n" + len(msg) + msg
        */
        return
            keccak256(
                abi.encodePacked("\x19Ethereum Signed Message:\n32", _messageHash)
            );
    }

    /* 4. 验证签名
    signer = 0xB273216C05A8c0D4F0a4Dd0d7Bae1D2EfFE636dd
    to = 0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C
    amount = 123
    message = "coffee and donuts"
    nonce = 1
    signature =
        0x993dab3dd91f5c6dc28e17439be475478f5635c92a56e17e82349d3fb2f166196f466c0b4e0c146f285204f0dcb13e5ae67bc33f4b888ec32dfe0a063e8f3f781b
    */
    function verify(
        address _signer,
        address _to,
        uint _amount,
        string memory _message,
        uint _nonce,
        bytes memory signature
    ) public pure returns (bool) {
        bytes32 messageHash = getMessageHash(_to, _amount, _message, _nonce);
        bytes32 ethSignedMessageHash = getEthSignedMessageHash(messageHash);

        return recoverSigner(ethSignedMessageHash, signature) == _signer;
    }

    function recoverSigner(bytes32 _ethSignedMessageHash, bytes memory _signature)
        public
        pure
        returns (address)
    {
        (bytes32 r, bytes32 s, uint8 v) = splitSignature(_signature);

        return ecrecover(_ethSignedMessageHash, v, r, s);
    }

    function splitSignature(bytes memory sig)
        public
        pure
        returns (
            bytes32 r,
            bytes32 s,
            uint8 v
        )
    {
        require(sig.length == 65, "invalid signature length");

        assembly {
            /*
            前 32 字节用于存储签名的长度

            add(sig, 32) = pointer of sig + 32
            跳过签名的前 32 字节
            
            mload(p) 从 memory 地址 p 开始，加载后面的 32 字节到 memory 中
            */

            // 第一段 32 字节
            r := mload(add(sig, 32))
            // 第二段 32 字节
            s := mload(add(sig, 64))
            // 最后一个字节 (下一段 32 字节中的第一个字节)
            v := byte(0, mload(add(sig, 96)))
        }

        // 可以显式 return (r, s, v) ，这里省略
    }
}
```



## 节约 Gas 的优化方法

- 将 `memory` 换为 `calldata`
- 将状态变量加载进 memory
- 将循环中的 i++ 换为 ++i
- 缓存数组元素
- 短路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.10;

// gas golf
contract GasGolf {
    // 初始状态（未进行优化时） - 50908 gas
    // 使用 calldata - 49163 gas
    // 将状态变量加载进 memory - 48952 gas
    // 短路 - 48634 gas
    // 修改循环计数增加方式 - 48244 gas
    // 缓存数组长度 - 48209 gas
    // load array elements to memory - 48047 gas

    uint public total;

    // 初始状态 - 未进行优化时
    // function sumIfEvenAndLessThan99(uint[] memory nums) external {
    //     for (uint i = 0; i < nums.length; i += 1) {
    //         bool isEven = nums[i] % 2 == 0;
    //         bool isLessThan99 = nums[i] < 99;
    //         if (isEven && isLessThan99) {
    //             total += nums[i];
    //         }
    //     }
    // }

    // gas 优化之后
    // [1, 2, 3, 4, 5, 100]
    function sumIfEvenAndLessThan99(uint[] calldata nums) external {
        uint _total = total;
        uint len = nums.length;

        for (uint i = 0; i < len; ++i) {
            uint num = nums[i];
            if (num % 2 == 0 && num < 99) {
                _total += num;
            }
        }

        total = _total;
    }
}
```

