##### && 是右值引用

左值的英文简写为“lvalue”，右值的英文简写为“rvalue”。很多人认为它们分别是"left value"、"right value" 的缩写，其实不然。lvalue 是“loactor value”的缩写，可意为存储在内存中、有明确存储地址（可寻址）的数据，而 rvalue 译为 "read value"，指的是那些可以提供数据值的数据（不一定可以寻址，例如存储于寄存器中的数据）



##### enum class

限定作用域的枚举类型(scoped enumeration)。定义限定作用域的枚举类型的一般形式是：首先是关键字[enum](https://so.csdn.net/so/search?q=enum&spm=1001.2101.3001.7020) class(或者等价地使用enum struct)，随后是枚举类型名字以及用花括号括起来的以逗号分隔的枚举成员(enumerator)列表，最后是一个分号。

枚举作用域(enumeration scope)是指枚举类型的成员的名字的作用域，起自其声明之处，终止枚举定义结束之处。C语言规定，枚举类型的成员(enumerator)的可见范围被提升至该枚举类型所在的作用域内。这被认为有可能污染了外部的作用域，为此，C++11引入了枚举类(enum class)解决此问题。