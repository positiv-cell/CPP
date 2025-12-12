#include <chrono>
#include <future>
#include <iostream>
#include <random>
#include <vector>

long long power(int base, int exp) {
    long long result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    return result;
}

int main() {
    std::vector<std::pair<int, int>> jobs = {{2, 30}, {3, 20}, {5, 15}, {7, 10}, {9, 12}};
    std::vector<std::future<long long>> futures;
    futures.reserve(jobs.size());

    auto start = std::chrono::steady_clock::now();
    for (auto [b, e] : jobs) {
        futures.emplace_back(std::async(std::launch::async, power, b, e));
    }
    for (size_t i = 0; i < jobs.size(); ++i) {
        auto [b, e] = jobs[i];
        std::cout << b << "^" << e << " = " << futures[i].get() << "\n";
    }
    auto end = std::chrono::steady_clock::now();
    std::cout << "Время: " << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count() << " мс\n";
    return 0;
}
