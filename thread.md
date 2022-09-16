# 线程类	thread



## 使用

```c++
//包含thread头文件
include <thread>
```

编译的时，链接这个库：libpthread.so





## 构造函数

```c++
// ①默认构造函，构造一个线程对象，在这个线程中不执行任何处理动作
thread() noexcept;

// ②移动构造函数，将 other 的线程所有权转移给新的 thread 对象。之后 other 不再表示执行线程
thread( thread&& other ) noexcept;

// ③创建线程对象，并在该线程中执行函数 f 中的业务逻辑，args 是要传递给函数 f 的参数任务函数 f 的可选类型有很多，具体如下：
//   普通函数，类成员函数，匿名函数，仿函数（这些都是可调用对象类型）
//   可以是可调用对象包装器类型，也可以是使用绑定器绑定之后得到的类型（仿函数）
template< class Function, class... Args >
explicit thread( Function&& f, Args&&... args );

// ④使用 =delete 显示删除拷贝构造，不允许线程对象之间的拷贝
thread( const thread& ) = delete;
```



## 成员函数

```c++
//get_id()
std::thread::id get_id() const noexcept;

//join()
void join();

//detach()
void detach();

//joinable()
bool joinable() const noexcept;

//operator=
// move (1)	
thread& operator= (thread&& other) noexcept;
// copy [deleted] (2)	
thread& operator= (const other&) = delete;
```





## e.g.

```c++
void func1(int num,string str);
void func2();

//构造函数
thread t1(func1,10,"abc");

thread t2;
t2 = thread(func2);

//成员函数
t1.get_id();
t1.join();
t1.detach();
t1.joinable();

//静态函数 thread::hardware_concurrency(),获取当前计算机的 CPU 核心数
cout<<"CPU number: "<<thread::hardware_concurrency()<<endl;
```



https://subingwen.cn/cpp/thread/

------



# 线程命名空间	this_thread



线程的命名空间std::this_thread,在这个命名空间中提供了四个公共的成员函数

```c++
//std::this_thread::get_id()
thread::id get_id() noexcept;

//std::this_thread::sleep_for();
template <class Rep, class Period>
  void sleep_for (const chrono::duration<Rep,Period>& rel_time);

//std::this_thread::sleep_until();
template <class Clock, class Duration>
  void sleep_until (const chrono::time_point<Clock,Duration>& abs_time);

//std::this_thread::yield();
void yield() noexcept;
```



https://subingwen.cn/cpp/this_thread/

------



# 线程同步之互斥锁



## 使用

```c++
//包含mutex头文件
#include <mutex>
```





- 找到多个线程操作的共享资源（全局变量、堆内存、类成员变量等），也可以称之为临界资源

- 找到和共享资源有关的上下文代码，也就是临界区（下图中的黄色代码部分）
- 在临界区的上边调用互斥锁类的 lock() 方法
- 在临界区的下边调用互斥锁的 unlock() 方法

线程同步的目的是让多线程按照顺序依次执行临界区代码，这样做线程对共享资源的访问就从并行访问变为了线性访问，访问效率降低了，但是保证了数据的正确性。



## std::mutex

```c++
//成员函数
//lock()
void lock();
void try_lock();

//unlock()
void unlock();
```



```c++
//e.g.
mutex m_mutex;

m_mutex.lock();
...
m_mutex.unlock();
```



## std::lock_gurad

lock_guard 在使用上面提供的这个构造函数构造对象时，会自动锁定互斥量，而在退出作用域后进行析构时就会自动解锁，从而保证了互斥量的正确操作，避免忘记 unlock() 操作而导致线程死锁

```c++
// 类的定义，定义于头文件 <mutex>
template< class Mutex >
class lock_guard;

// 常用构造函数
explicit lock_guard( mutex_type& m );
```



```c++
//e.g.
mutex m_mutex;

lock_guard<mutex> g_lock(m_mutex);
...
```



## std::recursive_mutex

递归互斥锁 std::recursive_mutex 允许同一线程多次获得互斥锁，可以用来解决同一线程需要多次获取互斥量时死锁的问题



虽然递归互斥锁可以解决同一个互斥锁频繁获取互斥锁资源的问题，但是还是建议少用，主要原因如下：

- 使用递归互斥锁的场景往往都是可以简化的，使用递归互斥锁很容易放纵复杂逻辑的产生，从而导致bug的产生

- 递归互斥锁比非递归互斥锁效率要低一些。

- 递归互斥锁虽然允许同一个线程多次获得同一个互斥锁的所有权，但最大次数并未具体说明，一旦超过一定的次数，就会抛出std::system错误

  

```c++
//e.g.
recursive_mutex m_mutex;

lock_gard<recursive_mutex> g_lock(m_mutex);
...
```



## std::timed_mutex

std::timed_mutex 是超时独占互斥锁，主要是在获取互斥锁资源时增加了超时等待功能，因为不知道获取锁资源需要等待多长时间，为了保证不一直等待下去，设置了一个超时时长，超时后线程就可以解除阻塞去做其他事情了。

std::timed_mutex 比 std::_mutex 多了两个成员函数：try_lock_for() 和 try_lock_until()



```c++
//成员函数
void lock();
bool try_lock();
void unlock();

// std::timed_mutex比std::_mutex多出的两个成员函数
template <class Rep, class Period>
  bool try_lock_for (const chrono::duration<Rep,Period>& rel_time);

template <class Clock, class Duration>
  bool try_lock_until (const chrono::time_point<Clock,Duration>& abs_time);
```



- try_lock_for 函数是当线程获取不到互斥锁资源的时候，让线程阻塞一定的时间长度

- try_lock_until 函数是当线程获取不到互斥锁资源的时候，让线程阻塞到某一个指定的时间点

- 关于两个函数的返回值：当得到互斥锁的所有权之后，函数会马上解除阻塞，返回 true，如果阻塞的时长用完或者到达指定的时间点之后，函数也会解除阻塞，返回 false

  

https://subingwen.cn/cpp/mutex/

------





# 条件变量



## 使用

```c++
//包含头文件
#include <condition_variable>
```



条件变量是 C++11 提供的另外一种用于等待的同步机制，它能阻塞一个或多个线程，直到收到另外一个线程发出的通知或者超时时，才会唤醒当前阻塞的线程。条件变量需要和互斥量配合起来使用，C++11 提供了两种条件变量：

- condition_variable：需要配合 `std::unique_lock<std::mutex>`进行 wait 操作，也就是阻塞线程的操作。

- condition_variable_any：可以和任意带有 lock()、unlock() 语义的 mutex 搭配使用，也就是说有四种：
- std::mutex：独占的非递归互斥锁
- std::timed_mutex：带超时的独占非递归互斥锁
- std::recursive_mutex：不带超时功能的递归互斥锁
- std::recursive_timed_mutex：带超时的递归互斥锁



条件变量通常用于生产者和消费者模型，大致使用过程如下：

1. 拥有条件变量的线程获取互斥量
2. 循环检查某个条件，如果条件不满足阻塞当前线程，否则线程继续向下执行
3. 产品的数量达到上限，生产者阻塞，否则生产者一直生产
4. 产品的数量为零，消费者阻塞，否则消费者一直消费
5. 条件满足之后，可以调用 notify_one() 或者 notify_all() 唤醒一个或者所有被阻塞的线程
6. 由消费者唤醒被阻塞的生产者，生产者解除阻塞继续生产
7. 由生产者唤醒被阻塞的消费者，消费者解除阻塞继续消费





## condition_variable


