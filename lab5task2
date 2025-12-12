#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>

class SimpleBarrier {
public:
    explicit SimpleBarrier(int parties) : parties(parties), count(parties) {}

    void arrive_and_wait() {
        std::unique_lock<std::mutex> lock(mtx);
        if (--count == 0) {
            count = parties;  // сброс для следующего этапа
            cv.notify_all();
        } else {
            cv.wait(lock, [this]() { return count == parties; });
        }
    }

private:
    int parties;
    int count;
    std::mutex mtx;
    std::condition_variable cv;
};

void stageWork(int id, int stage, int ms) {
    std::cout << "[Поток " << id << "] этап " << stage << "\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(ms));
}

int main() {
    const int threads = 4;
    SimpleBarrier sync(threads);

    std::vector<std::thread> pool;
    for (int i = 0; i < threads; ++i) {
        pool.emplace_back([i, &sync]() {
            stageWork(i, 1, 200 + i * 30);
            sync.arrive_and_wait();
            stageWork(i, 2, 150 + i * 20);
            sync.arrive_and_wait();
            stageWork(i, 3, 100 + i * 10);
        });
    }
    for (auto& t : pool) t.join();
    std::cout << "Все этапы выполнены.\n";
    return 0;
}
