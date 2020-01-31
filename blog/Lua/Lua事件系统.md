### Lua事件系统（EventManager.lua）

在使用LuaFrameWork的时候，苦于没有事件系统传递消息，于是写了这个事件管理器，进行多个模块之间、单个模块内部的消息传递。

作为事件管理器，那么就是需要管理事件，因此我选择function作为参数进行传递。事件需要一个唯一标识符，无论是数字还是string都可以，我选择string作为标识符，因为这样更容易辨识。事件有可能需要传递参数，lua的可变参数为 ‘...’三个点。

用string作为唯一标识符的问题在于可能会不小心出现两个同样的标识符，这样就会导致出现错误调用的问题，因此我添加了一个tag标签，用于区分不同模块的事件。



#### 保存事件

事件我用两个变量进行保存，一个是带tag的表示模块内部的事件容器 *eventsWithTag*，一个是不带tag的，表示通用的事件的容器 *events*

#### 添加事件

```
local function AddListener(list, name, func)
    if list[tostring(name)] == nil then
        list[tostring(name)] = {}
        table.insert(list[tostring(name)], func)
    else
        local eventList = list[tostring(name)]
        if not CheckContainFunc(eventList, func) then
            table.insert(eventList, func)
        else
            print(string.format("Events Contains A Same Key %s", name))
        end
    end
end

local function AddListenerWithTag(tag, name, func)
    if eventsWithTag[tostring(tag)] then
        local tagEvents = eventsWithTag[tostring(tag)]
        AddListener(tagEvents, name, func)
    else
        eventsWithTag[tostring(tag)] = {} -- add a new tag to keep events
        AddListener(eventsWithTag[tostring(tag)], name, func)
    end
end
```

tag 表示模块的唯一标识符，lua中没有过多的数据格式限制，如果需要的话，可以改为其他的数据格式，比如数字、table等等，管理器通过这个tag查找事件属于哪个模块的事件

name 表示事件的唯一标识符，管理器通过这个name查找对应的事件

func 表示回调函数，事件最终会调用这些方法，方法可以带参数，调用的时候如果不传递则为nil



#### 删除事件

```
local function RemoveListener(list, name, func)
    if list ~= nil then
        local eventList = list[tostring(name)]
        local index = CheckContainFunc(eventList, func)
        if index then
            print("Remove " .. name .. " Succeed !")
            table.remove(eventList, index)
        else
            print("Attempt To Remove A UnRegister Event: " .. name)
        end
    end
end

local function RemoveListenerWithTag(tag, name, func)
    local tmpEventTag = eventsWithTag[tostring(tag)]
    if tmpEventTag then
        RemoveListener(tmpEventTag, name, func)
    end
end
```

同样的，怎么添加事件就需要怎么样删除事件。通过tag跟name查找并删除即可。



#### 调用事件

```
local function DispatchListener(list, name, ...)
    if list[tostring(name)] ~= nil then
        local events = list[tostring(name)]
        for i, v in pairs(events) do
            if v ~= nil then
                v(...)
            end
        end
    else
        print(string.format("event is not registed %s", name))
    end
end

local function DispatchListenerByTag(tag, name, ...)
    local eventsList = eventsWithTag[tostring(tag)]
    DispatchListener(eventsList, name, ...)
end
```

调用的时候，根据传入的tag跟name查找事件，遍历name下所有的事件并调用。

具体方法可以看Scripts目录下的EventManager.lua文件，事件跟模块的唯一标识符最好也能够放在一个全局类中，这样模块之间调用事件不会出现不必要的交叉