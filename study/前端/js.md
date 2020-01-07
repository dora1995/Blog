1.内置类型：一共包含7中内置类型，6种基本类型（null, undefined, string, boolean, symbol, number）和对象（object）。其中数字类型number是浮点类型的（基于IEEE754标准），在使用中会有进度转换的问题

基本类型一般只是个字面量，只有在必要的时候才会转为对应的类型。对象是引用类型，在使用过程中有浅拷贝和深拷贝的问题

typeof：对于基本类型都能显示正确的类型，除了null会显示为object。对于对象，除了函数会显示function，其他都是显示object。（undefined, number, string, boolean, symbol, object, function）

可以使用Object.prototype.toString.call(xx)来获取正确的变量类型，会返回[Object Type]字符串

2.undefined不是保留字，在低版本浏览器可能可以被赋值，可以使用void 0（返回undefined）代替，而且代码量更少

3.类型转换：在条件判断是，除了undefined，null，false，NaN，''，0，-0外，其他的都是返回true，包括对象

对象在转换基本类型时，会先调动valueOf然后调用toString，两个方法都是可重写的。还有一个[Symbol.toPrimitive]，优先级最高

在四则运算中，加法运算中其中一方是字符串，就会把另一个也转为字符串。其他运算只要一方是数字，那么另一方就转为数字

==运算符：若两者类型相同，判断值是否相同。若两者类型不同，若有一方是number，将另一方调用ToNumber后再比较值；若有object，先调用ToPrimitive后再比较值。[] != [] // true

比较运算符：如果是对象，就通过toPromitive转换对象。如果是字符串，就通过unicode字符串索引来比较

4.原型：每个函数都有prototype属性，该属性指向原型。每个对象都有__proto__属性，指向了创建该对象的构造函数的原型。

new操作：生成一个对象、链接到原型、绑定this、返回新对象-