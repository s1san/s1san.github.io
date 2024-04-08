# 简单线程池实现

在编写应用程序时，我们为了更有效地利用系统资源并提高程序性能往往会自己实现一个线程池。尽管一些语言提供了多线程支持，但频繁的创建和销毁线程需要消耗大量时间。

我们可以事先创建一组线程，并且在任务执行完毕后不立即销毁，而是重复利用这些线程执行其他任务，从而避免了频繁的线程创建和销毁带来的开销。

线程池还可以限制系统中同时执行的线程数量，避免过度占用系统资源导致系统性能下降或崩溃。通过合理控制线程池中的线程数量，可以在保证系统稳定性的前提下最大程度地利用系统资源。

线程池组成部分：

* 线程集合（Workers）：一组预先创建的线程。
* 任务队列（Task Queue）：线程池中有一个任务队列，用于存储待执行的任务。当任务到达时，会被添加到任务队列中，等待线程池中的线程来执行。
* 互斥锁（Mutex）和条件变量（Condition Variable）：线程池使用互斥锁来保护任务队列的访问，以确保线程安全。条件变量用于线程之间的同步，当任务队列为空时，线程会等待条件变量的通知。
* 任务接口（Task Interface）：定义一个任务接口，用于将任务提交到线程池中执行。通常，任务接口是一个模板函数，可以接受任意类型的可调用对象，并返回一个 future 对象，用于获取任务的执行结果。
* 停止标志（Stop Flag）：线程池通常有一个标志变量，用于标识线程池是否停止。当线程池需要被销毁时，会将这个标志设置为 true，并通知所有线程退出执行。

本文基于C++11实现一个简单的线程池

### 定义线程池

```c++
class ThreadPool
{
public:
    ThreadPool(size_t threads = std::thread::hardware_concurrency());
    ~ThreadPool();

    template <class F, class... Args>
    auto enqueue(F &&f, Args &&...args)
        -> std::future<typename std::result_of<F(Args...)>::type>;

private:
    std::vector<std::thread> workers;    // 一组线程
    std::queue<std::function<void()>> tasks;    // 任务队列
    
    std::mutex queue_mutex;    // 互斥锁
    std::condition_variable condition;    // 条件变量
    bool stop;    // 停止标志
};
```

`std::thread::hardware_concurrency()`用于获取系统支持的并发线程数，它返回一个`unsigned int` 类型的值，表示当前系统中可以并行执行的最大线程数量，具体的值取决于硬件平台、操作系统和编译器等因素，可以作为线程池中线程数量的一个合理参考值。

`enqueue`函数将任务添加到线程池中并执行，`template <class F, class... Args>`声明一个变长模板，`F &&f` 表示可调用对象 `f` 的引用，`Args &&...args` 表示参数包，用于接收可调用对象 `f` 的参数列表。使用右值引用 `&&` 是为了实现[完美转发](https://zhuanlan.zhihu.com/p/369203981)，可以保持参数的值类别不变，同时支持左值和右值的传递。

随后指定模板函数的返回类型为 `std::future` 对象，用于获取任务的执行结果。 `typename std::result_of<F(Args...)>::type` 表示根据函数 `F` 和参数 `Args...` 推导出的结果类型。

### 构造函数实现

```c++
inline ThreadPool::ThreadPool(size_t threads)
    : stop(false)	// 将stop初始化为false，表示线程池初始状态为未停止状态
{
    for (size_t i = 0; i < threads; ++i)
    {
        workers.emplace_back(
            [this]
            {
                for (;;)    // 在 lambda 表达式内部，线程将会执行一个无限循环，不断地从任务队列中取出任务并执行。
                {
                    std::function<void()> task;    // 创建一个 std::function 对象 task，用于保存待执行的任务

                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock,
                                             [this]
                                             { return this->stop || !this->tasks.empty(); });
                        if (this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }

                    task();    // 执行任务
                }
            });
    }
}
```

在 lambda 表达式里面 for 循环的代码块中，使用 `std::unique_lock` 对 `queue_mutex` 进行加锁，确保线程安全地访问任务队列。再调用 `condition` 条件变量的 [wait](https://en.cppreference.com/w/cpp/thread/condition_variable/wait) 函数，阻塞线程，直到线程池停止运行或者任务队列非空。

如果线程池已经停止运行且任务队列为空，则退出循环，结束线程的执行。最后，将任务队列中的首个任务移动到 `task` 变量中，准备执行。

### `enqueue`函数实现

```c++
template <class F, class... Args>
auto ThreadPool::enqueue(F &&f, Args &&...args) -> std::future<typename std::result_of<F(Args...)>::type>
{
    // 使用 std::result_of 获取 F(Args...) 的返回类型，并命名为 return_type
    using return_type = typename std::result_of<F(Args...)>::type;
    
    auto task = std::make_shared<std::packaged_task<return_type()>>(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...));

    std::future<return_type> res = task->get_future();    // 调用 get_future() 函数，获取任务执行的结果
    {
        std::unique_lock<std::mutex> lock(queue_mutex);

        // 禁止在停止状态下添加任务。
        if (stop)
            throw std::runtime_error("enqueue on stopped ThreadPool");

        tasks.emplace([task]()
                      { (*task)(); });
    }

    condition.notify_one();
    
    return res;
}
```

`std::make_shared`接受一个模板参数`std::packaged_task<return_type()>`，并通过`std::bind`将可调用对象`f`和参数`args...`绑定到一起，生成一个函数对象来初始化`packaged_task`对象。该函数对象的类型是`std::packaged_task<return_type()>`，表示接受无参数并返回`return_type`类型的函数。

`std::forward`函数保持参数的完美转发，确保传递给`std::bind`的参数保持原始的引用类型。

### 析构函数实现

```c++
inline ThreadPool::~ThreadPool()
{
    {
        std::unique_lock<std::mutex> lock(queue_mutex);
        stop = true;
    }

    condition.notify_all();    // 通知所有线程，线程池已经停止运行
    for (std::thread &worker : workers)
        worker.join();
}
```

最后，在析构函数中，遍历所有线程，调用 `join()` 函数，等待线程执行完成，确保所有线程在退出之前都能正确地执行完毕。

### 测试代码实现

```c++
ThreadPool pool(4);    // 设线程池大小为4
std::vector<std::future<int>> results;

for (int i = 0; i < 16; ++i)
{
    results.emplace_back(
        pool.enqueue([i]
                     {
                         std::cout << "Task " << i << " started" << std::endl;
                         std::this_thread::sleep_for(std::chrono::seconds(1));
                         std::cout << "Task " << i << " finished" << std::endl;
                         return i * i; }));
}

for (auto &&result : results)
    std::cout << result.get() << ' ';
std::cout << std::endl;
```

本文代码在[这里](https://github.com/s1san/sbr/tree/main/SimpleThreadPool)

**references**

* Github, progschj, [ThreadPool](https://github.com/progschj/ThreadPool)