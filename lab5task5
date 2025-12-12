#include <condition_variable>
#include <functional>
#include <future>
#include <iostream>
#include <mutex>
#include <numeric>
#include <queue>
#include <thread>
#include <vector>

class ThreadPool {
public:
    explicit ThreadPool(size_t threads) {
        for (size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this]() { this->loop(); });
        }
    }

    template <class F, class... Args>
    auto submit(F&& f, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>> {
        using R = std::invoke_result_t<F, Args...>;
        auto task = std::make_shared<std::packaged_task<R()>>(std::bind(std::forward<F>(f), std::forward<Args>(args)...));
        std::future<R> res = task->get_future();
        {
            std::lock_guard<std::mutex> lock(mtx);
            if (stop) throw std::runtime_error("pool stopped");
            tasks.emplace([task]() { (*task)(); });
        }
        cv.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cv.notify_all();
        for (auto& t : workers) {
            if (t.joinable()) t.join();
        }
    }

private:
    void loop() {
        while (true) {
            std::function<void()> job;
            {
                std::unique_lock<std::mutex> lock(mtx);
                cv.wait(lock, [this]() { return stop || !tasks.empty(); });
                if (stop && tasks.empty()) return;
                job = std::move(tasks.front());
                tasks.pop();
            }
            job();
        }
    }

    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop = false;
};

unsigned long long factorial(int n) {
    if (n < 0) throw std::invalid_argument("n must be >= 0");
    unsigned long long r = 1;
    for (int i = 2; i <= n; ++i) r *= static_cast<unsigned long long>(i);
    return r;
}

long long sumRange(int from, int to) {
    long long sum = 0;
    for (int i = from; i <= to; ++i) sum += i;
    return sum;
}

void delayMs(int ms) {
    std::this_thread::sleep_for(std::chrono::milliseconds(ms));
}

int main() {
    size_t threads = std::max(2u, std::thread::hardware_concurrency());
    ThreadPool pool(threads);
    auto f1 = pool.submit(factorial, 12);
    auto f2 = pool.submit(sumRange, 1, 1'000'000);
    auto f3 = pool.submit(delayMs, 300);
    auto f4 = pool.submit(delayMs, 100);

    std::cout << "factorial(12)=" << f1.get() << "\n";
    std::cout << "sum 1..1e6=" << f2.get() << "\n";
    f3.get();
    f4.get();
    std::cout << "Задержки завершены\n";
    return 0;
}
