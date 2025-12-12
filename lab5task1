#include <chrono>
#include <iostream>
#include <numeric>
#include <random>
#include <thread>
#include <vector>

std::vector<double> movingAverageSequential(const std::vector<int>& data, int window) {
    std::vector<double> result(data.size(), 0.0);
    double sum = 0;
    for (size_t i = 0; i < data.size(); ++i) {
        sum += data[i];
        if (i >= static_cast<size_t>(window)) {
            sum -= data[i - window];
        }
        result[i] = sum / std::min<int>(window, static_cast<int>(i) + 1);
    }
    return result;
}

std::vector<double> movingAverageParallel(const std::vector<int>& data, int window, int threads) {
    std::vector<double> result(data.size(), 0.0);
    std::vector<std::thread> pool;
    pool.reserve(threads);
    size_t chunk = data.size() / threads;
    size_t rest = data.size() % threads;

    auto worker = [&](size_t start, size_t end) {
        double sum = 0;
        size_t prefixStart = (start > static_cast<size_t>(window)) ? start - window : 0;
        for (size_t i = prefixStart; i < start; ++i) sum += data[i];
        for (size_t i = start; i < end; ++i) {
            sum += data[i];
            if (i >= static_cast<size_t>(window)) sum -= data[i - window];
            result[i] = sum / std::min<int>(window, static_cast<int>(i) + 1);
        }
    };

    size_t pos = 0;
    for (int t = 0; t < threads; ++t) {
        size_t len = chunk + (t < static_cast<int>(rest) ? 1 : 0);
        size_t start = pos;
        size_t end = pos + len;
        pool.emplace_back(worker, start, end);
        pos = end;
    }
    for (auto& th : pool) th.join();
    return result;
}

int main() {
    const size_t n = 2'000'000;
    const int window = 50;
    int threads = std::max(2u, std::thread::hardware_concurrency());
    std::mt19937 gen(std::random_device{}());
    std::uniform_int_distribution<int> dist(0, 1000);
    std::vector<int> data(n);
    for (auto& v : data) v = dist(gen);

    auto t1 = std::chrono::steady_clock::now();
    auto seq = movingAverageSequential(data, window);
    auto t2 = std::chrono::steady_clock::now();
    auto par = movingAverageParallel(data, window, threads);
    auto t3 = std::chrono::steady_clock::now();

    std::cout << "Seq time: " << std::chrono::duration_cast<std::chrono::milliseconds>(t2 - t1).count() << " ms\n";
    std::cout << "Par time (" << threads << " threads): "
              << std::chrono::duration_cast<std::chrono::milliseconds>(t3 - t2).count() << " ms\n";
    std::cout << "Sample seq[1000]=" << seq[1000] << ", par[1000]=" << par[1000] << "\n";
    return 0;
}
