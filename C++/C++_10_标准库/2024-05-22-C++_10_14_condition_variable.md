---
category: Cpp
date: 2024-05-22 09:00:00 +0800
layout: post
title: C++_10_14_condition_variable
tag: CppSTL
---
## 简介

+ C++ <condition_variable>标准库

## condition_variable 介绍

+ 在C++11中，我们可以使用条件变量(condition_variable)实现多个线程间的同步操作；当条件不满足时，相关线程被一直阻塞，直到某种条件出现，这些线程才会被唤醒。

+ 条件变量是利用线程间共享的全局变量进行同步的一种机制，主要包括两个动作：
  + 一个线程因等待 条件变量的条件成立 而挂起
  + 另一个线程使 条件成立， 给出信号，从而唤醒被等待的线程。

+ 为了防止竞争，条件变量的使用总是和一个互斥锁结合在一起；通常情况下这个锁是std::mutex并且管理这个锁只能是 std::unique_lock<std::mutex> RAII模板类
+ 上面提到的两个步骤，分别是使用以下两个方法实现的：
  + 等待条件成立使用的是 condition_variable 类成员wait, wait_for 或 wait_util
  + 给出信号使用的是 condition_variable 类成员 notify_one 或者 notify_all 函数

## 使用说明

+ 在条件变量中只能使用 std::unique_lock<std::mutex> 说明

+ unique_lock 和 lock_guard 都是管理锁的辅助工具，都是RAII风格；它们是在定义时获得锁，在析构时释放锁。
+ 他们的主要区别在于 unique_lock 锁机制更加灵活。

## C++ <condition_variable>标准库

`<condition_variable>` 是 C++ 标准库中的头文件，提供了多线程编程中用于线程同步的条件变量。条件变量用于线程之间的同步和通信，允许一个线程在某个特定条件下等待或被唤醒，以进行进一步的处理。

条件变量通常与互斥量（mutex）结合使用，以便线程可以安全地等待某个条件并在条件满足时被通知。

下面是一个简单的示例，演示了如何使用 `<condition_variable>` 进行线程间的同步：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker_thread() {
    // 模拟一些工作
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_one(); // 通知等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

在这个示例中，主线程等待工作线程完成一些任务并准备好数据。它通过条件变量 `cv` 和互斥量 `mtx` 来实现同步和通信。主线程在互斥量的保护下等待条件满足，而工作线程在完成工作后发送信号通知主线程条件已满足。

`std::condition_variable` 允许线程在等待条件变为真时暂时阻塞，并在条件被满足时唤醒线程。它常与 `std::mutex` 和 `std::unique_lock` 一起使用，以确保线程安全并避免死锁。

条件变量是在多线程编程中实现线程同步的重要工具，它允许线程在特定条件下等待或被唤醒，并且能够准确地控制线程的执行流程。

## C++ <condition_variable>标准库 详解

`<condition_variable>` 是 C++ 标准库中提供的用于多线程编程的头文件之一，它包含了 `std::condition_variable` 类的定义，用于线程间的同步和通信。

### 主要成员函数和功能：

1. `wait()`：让线程等待直到满足某个特定条件。
    - 通过参数接受一个 `std::unique_lock` 对象和一个谓词（可以是 lambda 表达式），并在谓词返回 `true` 时解除阻塞，或者在收到通知时解除阻塞。
    - 在等待时会释放锁，并在被唤醒后再次获取锁，允许其他线程修改被保护的数据。

2. `notify_one()`：唤醒等待在条件变量上的一个线程。
    - 如果有多个线程在等待，只会唤醒其中一个线程（不保证唤醒的是哪个线程）。

3. `notify_all()`：唤醒等待在条件变量上的所有线程。
    - 唤醒所有等待的线程，允许它们竞争锁并继续执行。

4. `std::condition_variable` 是与互斥量（`std::mutex`）一起使用的，通过配合 `std::unique_lock` 对象，使线程能够在等待某个条件变为真时暂时阻塞自己，等待其他线程通知并且在条件满足时继续执行。

### 示例：

下面是一个简单的示例，演示了条件变量的基本用法：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool data_ready = false;

void worker_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        data_ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_one(); // 通知等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return data_ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

在此示例中，工作线程会等待一段时间模拟处理数据的操作，然后通知等待中的主线程。主线程在条件变量 `cv` 上等待，直到工作线程通知数据已准备就绪才会继续执行。

这个示例展示了如何使用 `std::condition_variable` 实现线程间的同步和通信，允许一个线程等待某个条件满足并在条件被满足时被通知。

## C++ <condition_variable>标准库 常用的类和函数

在C++中，`<condition_variable>` 标准库提供了 `std::condition_variable` 类，以及与之配合使用的一些常用函数和类型。

### 主要类和类型：

1. **std::condition_variable**：
   - 用于线程间的同步和通信。
   - 允许一个或多个线程等待某个条件满足，并在满足条件时被通知。
   - 主要成员函数有 `wait()`、`notify_one()`、`notify_all()`。
   - 配合 `std::mutex` 和 `std::unique_lock` 一起使用，允许线程在等待条件满足时暂时阻塞自己。

2. **std::mutex**：
   - 互斥量，用于保护共享资源，防止多个线程同时访问。
   - `std::unique_lock` 通常与 `std::mutex` 一起使用，用于在多线程环境中提供对共享资源的独占访问。

3. **std::unique_lock**：
   - 提供对互斥量的锁定和解锁操作。
   - 可以通过构造函数进行锁定并在作用域结束时自动解锁。

### 常用函数和成员方法：

1. **`wait(lock, pred)`**：
   - `std::condition_variable` 的成员函数。
   - 让线程等待，直到某个特定条件满足或者收到通知。
   - 参数 `lock` 是一个 `std::unique_lock` 对象（通常与互斥量一起使用），`pred` 是一个可调用对象，用于判断等待条件是否满足。

2. **`notify_one()`**：
   - `std::condition_variable` 的成员函数。
   - 唤醒一个等待在条件变量上的线程。

3. **`notify_all()`**：
   - `std::condition_variable` 的成员函数。
   - 唤醒所有等待在条件变量上的线程。

### 示例：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool data_ready = false;

void worker_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        data_ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_one(); // 通知等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return data_ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

这个示例展示了 `std::condition_variable` 如何与互斥量 `std::mutex` 结合使用，使得一个线程可以等待另一个线程满足某个条件后通知它。

## std::condition_variable

`std::condition_variable` 是 C++ 标准库 `<condition_variable>` 中定义的类，用于线程间的条件变量通信和同步。

它通常与 `std::mutex` 结合使用，实现线程的等待和唤醒操作，以在多线程环境中进行同步和通信。

### 主要操作和函数：

1. **wait()**：线程等待条件变量满足。在等待时，线程会释放与互斥量关联的锁，直到另一个线程通过 `notify_one()` 或 `notify_all()` 唤醒它。

    ```cpp
    std::condition_variable cv;
    std::mutex mtx;

    std::unique_lock<std::mutex> lck(mtx);
    cv.wait(lck); // 等待条件变量满足
    ```

2. **notify_one() 和 notify_all()**：用于唤醒一个或所有等待条件变量的线程。

    ```cpp
    std::condition_variable cv;
    std::mutex mtx;

    cv.notify_one(); // 唤醒一个等待条件变量的线程
    // 或
    cv.notify_all(); // 唤醒所有等待条件变量的线程
    ```

### 示例用法：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::condition_variable cv;
std::mutex mtx;
bool ready = false;

void workerThread() {
    std::unique_lock<std::mutex> lck(mtx);
    cv.wait(lck, [] { return ready; }); // 等待条件变量为 true

    std::cout << "Worker thread is processing..." << std::endl;
    // 执行工作...
}

int main() {
    std::thread worker(workerThread); // 创建新线程

    // 做一些其他工作...

    // 等待一段时间后，设置条件变量为 true
    std::this_thread::sleep_for(std::chrono::seconds(2));
    {
        std::lock_guard<std::mutex> lck(mtx);
        ready = true;
    }
    cv.notify_one(); // 唤醒等待的线程

    worker.join(); // 等待线程执行完毕

    return 0;
}
```

在这个示例中，`std::condition_variable` 被用于在工作线程等待一个条件变量为 `true` 的信号，主线程在一定条件下唤醒工作线程。工作线程在等待期间会释放锁，只有当条件满足时才会被唤醒继续执行。主线程通过 `notify_one()` 唤醒等待的工作线程。

## std::condition_variable::wait()

`std::condition_variable::wait()` 是 `std::condition_variable` 类的成员函数之一，用于让线程在条件变量上等待。

### 函数签名：
```cpp
void wait(std::unique_lock<std::mutex>& lock, Predicate pred);
```

### 参数：
- `lock`：一个 `std::unique_lock<std::mutex>` 对象的引用，用于保护等待期间可能修改的共享数据。
- `pred`：一个谓词（函数或函数对象），用于检查条件是否满足。如果谓词返回 `false`，`wait()` 函数将阻塞当前线程，并在收到通知或条件变为 `true` 时解除阻塞。

### 功能：
- 当前线程在调用 `wait()` 时会释放 `lock` 引用的互斥量，并进入阻塞状态，直到另一个线程调用 `notify_one()` 或 `notify_all()` 通知或条件满足，将当前线程从阻塞状态唤醒。
- 在线程被唤醒后，`wait()` 函数会重新获取 `lock` 引用的互斥量，并检查条件谓词。如果条件谓词返回 `true` 或被通知唤醒，`wait()` 函数返回，线程继续执行。

### 示例：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool data_ready = false;

void worker_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        data_ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_one(); // 通知等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return data_ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

在此示例中，`cv.wait(lock, pred)` 函数在等待期间会释放 `lock` 引用的互斥量，然后阻塞当前线程。当工作线程完成数据准备后，它会调用 `cv.notify_one()` 发送通知，唤醒主线程。主线程被唤醒后，重新获取互斥量，并检查 `data_ready` 变量，继续执行。

## td::condition_variable::notify_one()

`std::condition_variable::notify_one()` 是 `std::condition_variable` 类的成员函数之一，用于唤醒等待在条件变量上的一个线程。

### 函数签名：
```cpp
void notify_one() noexcept;
```

### 功能：
- `notify_one()` 用于通知等待在条件变量上的一个线程，唤醒其中一个被阻塞的线程。
- 如果有多个线程在条件变量上等待，只有其中一个线程会被唤醒。哪个线程会被唤醒是不确定的，取决于操作系统的调度和实现。

### 示例：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool data_ready = false;

void worker_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        data_ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_one(); // 通知等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return data_ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

在此示例中，`cv.notify_one()` 在工作线程中被调用，它唤醒了一个等待在条件变量 `cv` 上的主线程，使其从 `cv.wait()` 阻塞状态解除，并继续执行。需要注意的是，如果有多个线程在条件变量上等待，`notify_one()` 只会唤醒其中一个线程。

## std::condition_variable::notify_all()

`std::condition_variable::notify_all()` 是 `std::condition_variable` 类的成员函数之一，用于唤醒等待在条件变量上的所有线程。

### 函数签名：
```cpp
void notify_all() noexcept;
```

### 功能：
- `notify_all()` 用于通知等待在条件变量上的所有线程，唤醒所有被阻塞的线程。
- 调用 `notify_all()` 会使所有等待在条件变量上的线程都从等待状态解除，并尝试重新获取锁以继续执行。

### 示例：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool data_ready = false;

void worker_thread() {
    std::this_thread::sleep_for(std::chrono::seconds(1));

    {
        std::lock_guard<std::mutex> lock(mtx);
        data_ready = true;
        std::cout << "Worker: Data is ready." << std::endl;
    }

    cv.notify_all(); // 通知所有等待的线程
}

int main() {
    std::cout << "Main: Starting worker thread..." << std::endl;
    std::thread worker(worker_thread);

    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return data_ready; }); // 等待条件满足

        std::cout << "Main: Worker signaled data is ready." << std::endl;
    }

    worker.join(); // 等待工作线程结束

    return 0;
}
```

在此示例中，`cv.notify_all()` 在工作线程中被调用，它唤醒了所有等待在条件变量 `cv` 上的主线程。所有等待的线程都会从 `cv.wait()` 阻塞状态解除，并尝试重新获取锁以继续执行。`notify_all()` 会唤醒所有线程，让它们竞争锁资源并继续执行。

## C++ <condition_variable> std::condition_variable_any 类 详解

`std::condition_variable_any` 是 C++11 标准库中的一个同步原语，属于 `<condition_variable>` 头文件。它与 `std::condition_variable` 类似，但有所不同，主要在于它不要求条件变量与特定的锁类型（如 `std::mutex` 或 `std::unique_lock<std::mutex>`）绑定，因此更加灵活。

### 1. 基本概念
`std::condition_variable_any` 是一个条件变量，允许多个线程基于某个共享状态进行等待或通知。与 `std::condition_variable` 不同的是，它的等待操作不依赖于特定的互斥锁类型。它可以与任何类型的锁配合使用，只要该锁能够提供对共享数据的互斥访问。

### 2. `std::condition_variable_any` 和 `std::condition_variable` 的区别
- `std::condition_variable` 只能与 `std::unique_lock<std::mutex>` 这种特定的锁类型配合使用。
- `std::condition_variable_any` 可以与任何类型的锁（只要提供互斥性）一起使用，包括 `std::mutex`、`std::shared_mutex`、`std::timed_mutex` 等。

这使得 `std::condition_variable_any` 更加通用，但也要求用户更小心地管理锁的类型和行为。

### 3. 核心成员函数

- **`wait(std::unique_lock<Lock>& lock)`**  
  使当前线程在条件变量上等待，直到被通知并且 `lock` 被重新获得。`lock` 必须是一个 `std::unique_lock`，并且它的锁类型可以是任何提供互斥的类型。

- **`wait_for(std::unique_lock<Lock>& lock, std::chrono::duration<Rep, Period> const& rel_time)`**  
  与 `wait` 类似，但是它会在给定的时间段内超时。如果在超时之前没有被通知，线程会自动继续执行。

- **`wait_until(std::unique_lock<Lock>& lock, std::chrono::time_point<Clock, Duration> const& abs_time)`**  
  等待直到某个特定的时间点，或者直到通知为止。

- **`notify_one()`**  
  通知一个等待中的线程。如果有多个线程在等待条件变量，只有一个会被唤醒。

- **`notify_all()`**  
  通知所有等待中的线程。

### 4. 使用示例

```cpp
#include <iostream>
#include <thread>
#include <condition_variable>
#include <mutex>

std::condition_variable_any cv;
std::mutex mtx;
bool ready = false;

void print_id(int id) {
    std::unique_lock<std::mutex> lck(mtx);
    while (!ready) {
        cv.wait(lck);  // Wait until ready is true
    }
    std::cout << "Thread " << id << '\n';
}

void go() {
    std::unique_lock<std::mutex> lck(mtx);
    ready = true;
    cv.notify_all();  // Notify all waiting threads
}

int main() {
    std::thread threads[10];
    
    // Launch threads
    for (int i = 0; i < 10; ++i) {
        threads[i] = std::thread(print_id, i);
    }
    
    std::cout << "Preparing to launch threads...\n";
    go();  // Notify all threads to proceed
    
    for (auto& th : threads) {
        th.join();
    }

    return 0;
}
```

### 5. `std::condition_variable_any` 的使用场景
`std::condition_variable_any` 更适用于那些需要灵活选择锁类型的场景。比如，你可能使用 `std::shared_mutex` 来允许多个线程共享读访问，但仍然需要在某些时刻进行写访问控制。在这种情况下，`std::condition_variable_any` 会比 `std::condition_variable` 更适用。

### 6. 注意事项
- **性能考量**：`std::condition_variable_any` 在某些情况下可能会有更多的性能开销，因为它需要在底层提供更多的灵活性来支持不同类型的锁。
- **锁的类型匹配**：在使用时，你需要确保传入的锁类型能够正确地支持 `wait`、`wait_for` 和 `wait_until` 等操作。

总结来说，`std::condition_variable_any` 提供了比 `std::condition_variable` 更强的灵活性，但也要求开发者更细致地考虑锁的类型和同步的复杂性。如果你只需要与 `std::mutex` 配合使用，通常使用 `std::condition_variable` 会更加简单和高效。