#include <algorithm>
#include <chrono>
#include <iostream>
#include <random>
#include <thread>
#include <vector>

template <typename It>
void merge(It begin, It mid, It end) {
    std::vector<typename It::value_type> tmp;
    tmp.reserve(std::distance(begin, end));
    It i = begin, j = mid;
    while (i != mid && j != end) {
        if (*i < *j) tmp.push_back(*i++);
        else tmp.push_back(*j++);
    }
    tmp.insert(tmp.end(), i, mid);
    tmp.insert(tmp.end(), j, end);
    std::move(tmp.begin(), tmp.end(), begin);
}

void parallelMergeSort(std::vector<int>& arr, int depth = 0) {
    if (arr.size() < 1000 || depth > 3) {
        std::sort(arr.begin(), arr.end());
        return;
    }
    auto midIter = arr.begin() + arr.size() / 2;
    std::vector<int> left(arr.begin(), midIter);
    std::vector<int> right(midIter, arr.end());

    std::thread t([&left, depth]() { parallelMergeSort(left, depth + 1); });
    parallelMergeSort(right, depth + 1);
    t.join();
    std::merge(left.begin(), left.end(), right.begin(), right.end(), arr.begin());
}

int main() {
    const size_t n = 200000;
    std::vector<int> data(n);
    std::mt19937 gen(std::random_device{}());
    std::uniform_int_distribution<int> dist(-100000, 100000);
    for (auto& v : data) v = dist(gen);

    auto copy = data;
    auto t1 = std::chrono::steady_clock::now();
    std::sort(copy.begin(), copy.end());
    auto t2 = std::chrono::steady_clock::now();

    auto par = data;
    auto t3 = std::chrono::steady_clock::now();
    parallelMergeSort(par);
    auto t4 = std::chrono::steady_clock::now();

    std::cout << "std::sort: " << std::chrono::duration_cast<std::chrono::milliseconds>(t2 - t1).count() << " мс\n";
    std::cout << "parallel merge sort: " << std::chrono::duration_cast<std::chrono::milliseconds>(t4 - t3).count() << " мс\n";
    std::cout << "Первые элементы: std=" << copy[0] << ", par=" << par[0] << "\n";
    return 0;
}
