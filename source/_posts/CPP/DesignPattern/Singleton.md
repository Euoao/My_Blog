---
title: 【初探设计模式】——单例模式
tags:
  - 设计模式
  - 单例模式
categories:
  - 设计模式
abbrlink: e7a7bc11
---

**单例模式**是一种创建型设计模式， 保证一个类只有一个实例， 并提供一个访问该实例的全局节点。

<!-- more -->

有以下几个要点

1. 全局只有一个实例，使用 `static` 特性实现，并将构造函数设为私有避免在外部构造
2. 通过共有接口获得实例
3. 线程安全
4. 禁止拷贝和赋值

# 实现方式

有**懒汉式**和**饿汉式**两种实现方式，两者区别在于创建实例的时间不一样，懒汉式在首次使用实例时创建实例（需要注意线程安全），饿汉式在系统一运行时创建实例（本身保证线程安全），更推荐前者。

## 懒汉式

C++11之后，可以通过返回静态局部变量引用来实现

``` cpp
// 懒汉式实现
class Singleton {
public:
    // 删除拷贝构造和拷贝赋值函数 或者 设置为 private，禁止外部拷贝赋值
    Singleton(const Singleton& singleton) = delete;

    Singleton operator = (const Singleton& singleton) = delete;

    static Singleton& GetInstance() {
        static Singleton singleton;
        return singleton;
    }

private:
    // 将构造函数设置为 private，禁止外部构造和析构
    Singleton() {}

    ~Singleton() {}
};
```

这利用了C++11的一个特性：如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。

另一种实现方法是使用锁、共享指针、互斥量来达到线程安全

``` cpp
class Singleton {
public:

    Singleton() {}

    ~Singleton() {}

    Singleton(const Singleton& singleton) = delete;

    Singleton operator = (const Singleton& singleton) = delete;

    static std::shared_ptr<Singleton> GetInstance() {
        // "double checked lock"
        if (m_ptr == nullptr) {
            std::lock_guard<std::mutex> lk(m_mutex);
            if (m_ptr == nullptr) {
                m_ptr = std::make_shared<Singleton>();
            }
        }
        return m_ptr;
    }


private:
    static std::shared_ptr<Singleton> m_ptr;
    static std::mutex m_mutex;
};
std::shared_ptr<Singleton> Singleton::m_ptr = nullptr;
std::mutex Singleton::m_mutex;
```

这里使用双检锁技术，即用两个`if`判断语句，只有判断指针为空的时候才加锁，避免每次调用 `GetInstance()`的方法都加锁。

## 饿汉式

**饿汉式**即运行时初始化实例，缺点是容易产生垃圾对象

``` cpp
// 饿汉式实现
class Singleton {
public:
    Singleton(const Singleton& Singleton) = delete;

    Singleton operator = (const Singleton& Singleton) = delete;

    static Singleton& GetInstance() {
        return m_Singleton;
    }


private:
    Singleton() {}

    ~Singleton() {}

    static Singleton m_Singleton;
};

Singleton Singleton::m_Singleton;
```
## 区别

下面一个实例可以明显的看出二者区别

``` cpp
#include <iostream>
#include <mutex>
#include <memory>

namespace  Lazybones {

// 懒汉式实现
class Singleton {
public:

    Singleton(const Singleton& singleton) = delete;

    Singleton operator = (const Singleton& singleton) = delete;

    static Singleton& GetInstance() {
        static Singleton singleton;
        return singleton;
    }


private:
    Singleton() {
        std::cout << "Create a lazybones singleton" << std::endl;
    }

    ~Singleton() {
        std::cout << "Derive a lazybones singleton" << std::endl;
    }
};

}

namespace Hungry{
// 饿汉式实现
class Singleton {
public:
    Singleton(const Singleton& Singleton) = delete;

    Singleton operator = (const Singleton& Singleton) = delete;

    static Singleton& GetInstance() {
        return m_Singleton;
    }


private:
    Singleton() {
        std::cout << "Create a hungry singleton" << std::endl;
    }

    ~Singleton() {
        std::cout << "Derive a hungry singleton" << std::endl;
    }

    static Singleton m_Singleton;
};

Singleton Singleton::m_Singleton;
}
int main() {
    std::cout << "------\n";
{
    std::cout << "------\n";
    Lazybones::Singleton::GetInstance();
    Hungry::Singleton::GetInstance();
}
    std::cout << "------\n";
}
```
以下是该程序结果

```
Create a hungry singleton
------
------
Create a lazybones singleton
------
Derive a lazybones singleton
Derive a hungry singleton
```

饿汉式在一开始类加载的时候就实例化，无论使用与否，都会实例化；懒汉式什么时候用就什么时候实例化。