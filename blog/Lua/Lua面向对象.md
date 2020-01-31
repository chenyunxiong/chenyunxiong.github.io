### Lua面向对象

Lua本身是没有面向对象这个概念的，但是table的各种用法却可以模拟出类似面向对象的东西。Lua中唯一的数据结构就是Table，那么自然，一个类自然也就是一个Table。

```
Person = {
    age = 10,
    name  = "ee";
    Init = function(age, name)
        Person.age = age;
        Person.name = name;
    end
}

print(Person.age, Person.name) -- 输出 10 "ee"
Person.Init(1, "面向对象")
print(Person.age, Person.name) -- 输出 1 "面向对象"
```

如上，Person就是一张简单的Table，也可以看作一个基本类，拥有age跟name两个字段，Init方法。直接以Person开头调用表中的方法跟字段

当然，这只是简单的类的模拟，只能有一个，而无法以Person为模板，创建多个Person实例。这时候就需要用到Lua的元表（mateTable）跟元方法(metaMethod)。



#### 元表与元方法

元表的作用在于以一个表作为模板，使对象克隆多个实例成为可能。元表可以使关联该元表生成的表都可以共享元表的方法跟属性。而原方法的作用在于，可以使Table表之间的操作变的更自由跟定制。将上面的Person进行修改如下：

```
function Person:New(age, name)
    local o = {}
    setmetatable(o, self)
    self.__index = self
    o.age = age
    o.name = name
    return o
end

function Person:Print()
    print(self.age, self.name)
end

local p = Person:New( 2, "tt3")
p:Print()
local p2 = Person:New( 3, "ttt")
p:Print() -- 输出 2 tt3
p2:Print() -- 输出 3 ttt
print(getmetatable(p), Person) -- 输出两个同样的数值
```

setmetatable(t, mt) 表示将一个表 t 的元表关联为 mt，这样每一个调用New方法以后，返回的表都是关联的同一个元表，仅仅是这样还不够，返回的表虽然已经关联到元表Person，但也只是关联了而已，新生成的 p 跟 p2 还是无法调用Person中的方法，因为这是表与表之间的操作。

此时就需要用到原方法。__index 元方法定义了元表与关联便之间的关系：如果关联表中自己找得到调用的方法跟字段的 话，则调用关联表中的。如果关联表中找不到的方法跟字段，则会去元表中查找对应的方法跟字段，如果有则调用。没有则提示不存在的方法跟字段

如此，当调用 p:Pring() 跟 p2:Pring() 的时候，p跟p2中其实并没有Print()方法，但是还是正确的输出了，正是调用了元表的Print()方法。这也就正确的模拟了对象生成实例的过程。

#### 类的继承

面向对象的一大特点就是继承，Lua的元表跟元方法也能模拟，Teacher继承了Person，并重写了Print()方法。

```
Person = {}

function Person:New(age, name)
    local o = {}
    setmetatable(o, self)
    self.__index = self
    o.age = age
    o.name = name

    return o
end

function Person:GetAge()
    return self.age
end

function Person:Print()
    print(self.age, self.name)
end

Teacher = Person:New()
function Teacher:New(age, name, course)
    local t = Person:New(age, name)
    setmetatable(t, self)
    self.__index = self
    t.course = course
    return t
end

function Teacher:Print()
    print(self.age, self.name, self.course)
end

local t1 = Teacher:New(33, "王老师", "语文")
t1:Print() -- 输出 33 王老师 语文
print("age:", t1:GetAge()) -- 输出 age:33
```

Teacher并没有重写GetAge()方法，然后 t1:GetAge() 却正确的输出了，可见是正确的继承了这个方法。而重写的Print()方法，多输出了一个课程，也正常输出，可见重写也是可以的。



#### 方法访问权限（私有公有）

冒号的作用是省略了self的传递，也就是说，如果我们不想写冒号改为写等号的话，那么每个方法的第一个参数必然是self：

```
function Person.Print()
end

function Person:Console()
end

p = Person:New()
p.Print(self)
p:Console()
```

可以看上图的Print方法跟Console方法，self是为了找到对应的表中的方法，不传的话，必然会报错。冒号相当于语法糖，可以忽略self的传递，使得方法更加简洁明了。

以上两种方法都可以说是Public方法，实例p是可以正常访问到的。



私有方法同样需要传递这个self，区别在于，私有方法是用local写的。跟其他语言不同之处在于，Lua中模拟的私有方法并没有确定的归属，换句话说，它只属于其所写的Lua文件，而不是写在文件中的某个Table表，当然只要控制号一个文件只写一个类则不存在这个问题。

```
local function Console(self)  -- 私有方法 Console
    print(self.age, self.name, self.course)
end

local t1 = Teacher:New(33, "王老师", "语文")
Console(t1)  -- 输出为：33 王老师  语文
local p = Person:New( 2, "tt3")
Console(p) -- 输出为：2 tt3 nil
```

可以看出来，Console是私有方法，外部无法访问。但是Person跟Teacher都可以访问，也就是说Console方法既不属于Person也不属于Teacher，它只管你输入的self是谁，但是如果我们只写了一个Person方法，而没有写继承的Teacher，则不用考虑这个问题。

