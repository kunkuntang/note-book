```
function one<T>(a: T[]): T {
    return a[0];
}

let two: <T>(a: T[]) => T = function (a) {
    return a[0];
}

// 接口 （调用时才推断）
interface some {
    <T>(a: T[]): T
}

// 接口泛型 （定义时推断）
interface ISome<T> {
    (a: T[]): T
}

let three: some = function (a) {
    return a[0];
}

let four: ISome<number> = function (a) {
    return a[0]
}

```