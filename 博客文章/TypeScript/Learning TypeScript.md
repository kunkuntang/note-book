# 使用函数

## 在TypeScript中使用函数

### 函数声明和函数表达式

在JavaScript中有两种声明函数形式：命名函数和匿名函数。对于匿名函数来说，也有我们可以把它赋值给一个变量，从而变得与命名函数的表现一致。
```JavaScript
function namedFunc() {} //命名函数
var unnamedFunc = function() {} // 把一个匿名函数赋值给一个变量
```

> 它们两的不同之处在于**变量提升**

### 函数类型

```
function greetNamed(name : string) : string {
    if (name) {
        return 'Hi!' + name;
    }
}
```

声明了一个参数为字符串的，返回值为字符串的命名函数。

先声明一个变量，再把一个函数赋值给这个变量：
```
var greeetUnnamed : (name : string) => string;

greeetUnnamed = function(name : string) : string {
    if (name) {
        return 'Hi!' + name;
    }
}
```
合成一行的写法：
```
var greeetUnnamed : (name : string) => string = function(name : string) : string {
    if (name) {
        return 'Hi!' + name;
    }
}
```

### 有可选参数的函数

```
function add(foo : number, bar : number, foobar? : number) : number {
    var result = foo + bar;
    if(foobar !== undefined) {
        result += foobar;
    }
    return result;
}
```

参数foobar的名称后面跟着一个`?`，说明了这个参数不是必填的。当我们提供两个或三个参数时编译器不会报错。

```
add() // 报错
add(2, 2) // 4
add(2, 2, 2) // 6
```
### 有默认参数的函数

```
function add(foo : number, bar : number, foobar : number = 0) : number {
    var result = foo + bar;
    if(foobar !== undefined) {
        result += foobar;
    }
    return result;
}
```

参数foobar在声明类型后面使用`=`操作符并提供一个默认值，即可指定这个参数是可选的，并且在foobar没有传给函数时设置一个默认值。

### 有剩余参数的函数

```
function add(...foo : number[]) : number {
    var result = 0;
    for(var i = 0; i < foo.length; i++) {
        result += foo[i]
    }
    return result;
}
```

上面代码使用foo参数代替了之前的foo、bar和foobar，并且foo参数前面有三个点的省略号。一个剩余参数必须包含一个数组类型，否则会出现编译错误。现在可以以任意数量调用add函数。

```
add() // 0
add(2) // 2
add(2, 2) // 4
add(2, 2, 2) // 6
```

虽然没有具体的参数数量限制，天蝎座上可以取数字类型的最大值。但实际上，这依赖于如何调用这个函数。

### 函数重载

函数重载或方法是使用**相同名字和不同参数或类型**创建多个方法的一种能力。

要实现函数重载
 - 先要声明所有重载方法的定义，**注意不包含方法的实现**。
 - 再声明一个参数为any类型或联合类型的实现方法。
 - 实现实现方法并通过参数类型（和返回类型）不同来实现重载。
 
```
// 先要声明所有重载方法的定义
function test(name: string): string; // 重载方法
function test(age: number): string; // 重载方法
function test(single: boolean): string; // 重载方法
// 声明一个参数为any类型或联合类型的实现方法
function test(value: (string | number | boolean): string { // 实现方法
    switch(typeof value) {
        case 'string': return `My name is ${vallue}`;
        case 'number': return `I am ${vallue} years old`;
        case 'boolean': return value ? "I am single." : "I am not single.";
    }
}
```

> 实现方法必须在所有重载方法的最后面

### 范型

先看下面例子：
```
class User {
    name: string;
    age: number;
}

function getUsers(cb: (users: User[]) => void): void {
    $.ajax({
        url: '/api/users',
        method: 'GET',
        success: function(data) {
            cb(data.items);
        },
        error: function(error) {
            cb(null);
        }
    })
}

class Order {
    id: number;
    total: number;
    items: any[];
}
function getOrders(cb: (orders: Order[]) => void): void {
    $.ajax({
        url: '/api/orders',
        method: 'GET',
        success: function(data) {
            cb(data.items);
        },
        error: function(error) {
            cb(null);
        }
    })
}
```

上面代码中getOrders函数和getUsers函数几乎完全相同，其中不同的只有请求url和传入cb的参数。

面对以上场景我们可以使用泛型来避免代码重复。

```
class User {
    name: string;
    age: number;
}

class Order {
    id: number;
    total: number;
    items: any[];
}

function getEntities<T>(url: string, cb: (list: T[]) => void): void {
    $.ajax({
        url: url,
        method: 'GET',
        success: function(data) {
            cb(data.items);
        },
        error: function(error) {
            cb(null);
        }
    })
}
```

在函数名后面增加一对括号（< >）来表明这是一个范型函数。如果角括号内是一个字符T，则它表示一种类型。函数的第一个参数是字符串类型的url，第二个参数是函数类型的cb，它接受一个T类型的参数作为唯一的参数。我们可以这样使用函数：

```
function getEntities<User>('/api/users', function(users: User[]) {
    for(var i = 0; users.length; i++) {
        console.log(users[i].name);
    }
}

function getEntities<Order>('/api/orders', function(orders: Order[]) {
    for(var i = 0; orders.length; i++) {
        console.log(orders[i].total);
    }
}
```

## TypeScript中的异步编程

### 回调和高阶函数


