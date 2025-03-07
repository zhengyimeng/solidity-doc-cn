.. index:: ! operator

操作符
=========

即使两个操作数的类型不一样，也可以进行算术和位操作运算。
例如，你可以计算 ``y = x + z`` ，其中 ``x`` 是 ``uint8`` ， ``z`` 是 ``int32`` 类型。
在这些情况下，将使用以下机制来确定运算结果的类型（这在溢出的情况下很重要）。


1. 如果右操作数的类型可以隐含地转换为左操作数的类型的类型，则使用左操作数的类型。
2. 如果左操作数的类型可以隐含地转换为右操作数的类型的类型，则使用右操作数的类型。
3. 否则，该操作不被允许。

如果其中一个操作数是一个 :ref:`常量数字<rational_literals>`，会首先被转换为能容纳该值的最小的类型 (相同位数时，无符号类型被认为比有符号类型 "小")。
如果两者都是常量数字，则以任意的精度进行计算。


操作符的结果类型与执行操作的类型相同，除了比较运算符，其结果总是 ``bool``。

运算符 ``**``（幂）， ``<<``  和 ``>>`` 使用左边操作数的类型来作为运算结果类型。


三元运算符
----------------

三元运算符是一个表达是形式： ``<expression> ? <trueExpression> : <falseExpression>`` 。
它根据 ``<expression>`` 的执行结果，选择后两个给定表达式中的一个。
如果 ``<expression>`` 执行结果 ``true`` ，那么 ``<trueExpression>`` 将被执行，否则 ``<falseExpression>`` 被执行。

三元运算符的结果不会为有理数类型，即使它的所有操作数都是有理数类型。
结果类型是由两个操作数的类型决定的，方法与上面一样，如果需要的话，首先转换为它们的最小可容纳类型（mobile type ）。

因此， ``255 + (true ? 1 : 0)`` 将由于算术溢出而被回退。
原因是 ``(true ? 1 : 0)`` 是 ``uint8`` 类型，这迫使加法也要在 ``uint8`` 中执行。
而256超出了这个类型所允许的范围。

另一个结果是，像 ``1.5 + 1.5`` 这样的表达式是有效的，但 ``1.5 + (true ? 1.5 : 2.5)`` 则无效。
这是因为前者是以无限精度来进行有理表达式运算，只有它的最终结果值才是重要的。
后者涉及到将小数有理数转换为整数，这在目前是不允许的。


.. index:: assignment, lvalue, ! compound operators

复合操作及自增自减操作
--------------------------------------------

如果 ``a`` 是一个 LValue（即一个变量或者其它可以被赋值的东西），以下运算符都可以使用简写：

``a += e`` 等同于 ``a = a + e``。其它运算符如 ``-=``， ``*=``， ``/=``， ``%=``， ``|=``， ``&=`` ， ``^=`` ， ``<<=`` 和 ``>>=``  都是如此定义的。
``a++`` 和 ``a--`` 分别等同于 ``a += 1`` 和 ``a -= 1``，但表达式本身的值等于 ``a`` 在计算之前的值。
与之相反， ``--a`` 和 ``++a`` 虽然最终 ``a`` 的结果与之前的表达式相同，但表达式的返回值是计算之后的值。

.. index:: !delete
.. _delete:

delete
----------

``delete a`` 的结果是将 ``a`` 类型初始值赋值给 ``a``。即对于整型变量来说，相当于 ``a = 0``，delete 也适用于数组，对于动态数组来说，是将重置为数组长度为0的数组，而对于静态数组来说，是将数组中的所有元素重置为初始值。对数组而言， ``delete a[x]`` 仅删除数组索引 ``x`` 处的元素，其他的元素和长度不变，这以为着数组中留出了一个空位。如果打算删除项，映射可能是更好的选择。

如果对象  ``a``  是结构体，则将结构体中的所有属性(成员)重置。 

换句话说，在 ``delete a`` 之后 ``a`` 的值与在没有赋值的情况下声明 ``a`` 的情况相同，
但需要注意以下几点：

``delete`` 对整个映射是无效的（因为映射的键可以是任意的，通常也是未知的）。
因此在你删除一个结构体时，结果将重置所有的非映射属性（成员），这个过程是递归进行的，除非它们是映射。
然而，单个的键及其映射的值是可以被删除的。

理解 ``delete a`` 的效果就像是给 ``a`` 赋值很重要，换句话说，这相当于在 ``a`` 中存储了一个新的对象。

当 ``a`` 是应用变量时，我们可以看到这个区别， ``delete a`` 它只会重置 ``a`` 本身，而不是更改它之前引用的值。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract DeleteLBC {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // 将 x 设为 0，并不影响数据
            delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
            uint[] storage y = dataArray;
            delete dataArray; 
            // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
            // 因为它是一个存储位置是 storage 的对象的别名。
            // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
            assert(y.length == 0);
        }
    }