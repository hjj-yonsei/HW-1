#include <iostream>
#include <vector>
#include <chrono>
#include <algorithm>
#include <random>
#include <iomanip>

using namespace std;
using namespace std::chrono;

const int RUN = 32;

// ==========================================
// 1. 버블 정렬 (Bubble Sort) - O(N^2)
// ==========================================
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    bool swapped;
    for (int i = 0; i < n - 1; i++) {
        swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}

// ==========================================
// 2. 퀵 정렬 (Quick Sort) - O(N log N)
// ==========================================
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high]; // Lomuto 파티션 (가장 오른쪽 값을 피벗으로)
    int i = (low - 1);
    for (int j = low; j <= high - 1; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return (i + 1);
}

void quickSort(vector<int>& arr, int low, int high) {
    // 스택 오버플로우 방지를 위한 간단한 깊이 제한 기법이 실무에선 쓰이나,
    // 본 과제에서는 O(N^2) 최악의 경우를 관찰하기 위해 기본형을 사용합니다.
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

void quickSortWrapper(vector<int>& arr) {
    quickSort(arr, 0, arr.size() - 1);
}

// ==========================================
// 3. 팀 정렬 (Tim Sort) - O(N log N)
// ==========================================
// Tim Sort를 위한 삽입 정렬 부분
void insertionSort(vector<int>& arr, int left, int right) {
    for (int i = left + 1; i <= right; i++) {
        int temp = arr[i];
        int j = i - 1;
        while (j >= left && arr[j] > temp) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = temp;
    }
}

// Tim Sort를 위한 병합 부분
void merge(vector<int>& arr, int l, int m, int r) {
    int len1 = m - l + 1, len2 = r - m;
    vector<int> left(len1), right(len2);
    for (int i = 0; i < len1; i++) left[i] = arr[l + i];
    for (int i = 0; i < len2; i++) right[i] = arr[m + 1 + i];

    int i = 0, j = 0, k = l;
    while (i < len1 && j < len2) {
        if (left[i] <= right[j]) arr[k++] = left[i++];
        else arr[k++] = right[j++];
    }
    while (i < len1) arr[k++] = left[i++];
    while (j < len2) arr[k++] = right[j++];
}

void timSort(vector<int>& arr) {
    int n = arr.size();
    // 1. Minrun 크기 단위로 삽입 정렬 수행
    for (int i = 0; i < n; i += RUN)
        insertionSort(arr, i, min((i + RUN - 1), (n - 1)));

    // 2. 크기를 2배씩 늘려가며 병합 수행
    for (int size = RUN; size < n; size = 2 * size) {
        for (int left = 0; left < n; left += 2 * size) {
            int mid = left + size - 1;
            int right = min((left + 2 * size - 1), (n - 1));
            if (mid < right) merge(arr, left, mid, right);
        }
    }
}

// ==========================================
// 데이터 생성 유틸리티
// ==========================================
vector<int> generateRandom(int n) {
    vector<int> arr(n);
    mt19937 gen(42); // 고정 시드로 재현성 확보
    uniform_int_distribution<int> dis(1, n * 10);
    for (int i = 0; i < n; i++) arr[i] = dis(gen);
    return arr;
}

vector<int> generateReversed(int n) {
    vector<int> arr(n);
    for (int i = 0; i < n; i++) arr[i] = n - i;
    return arr;
}

vector<int> generateNearlySorted(int n) {
    vector<int> arr(n);
    for (int i = 0; i < n; i++) arr[i] = i + 1;
    // 5%의 데이터만 무작위 셔플
    mt19937 gen(42);
    uniform_int_distribution<int> dis(0, n - 1);
    for (int i = 0; i < n * 0.05; i++) {
        int idx1 = dis(gen), idx2 = dis(gen);
        swap(arr[idx1], arr[idx2]);
    }
    return arr;
}

// ==========================================
// 성능 측정 래퍼 함수
// ==========================================
double measureTime(void (*sortFunc)(vector<int>&), vector<int> arr) {
    auto start = high_resolution_clock::now();
    sortFunc(arr);
    auto end = high_resolution_clock::now();
    
    // 밀리초(ms) 단위로 반환
    return duration<double, milli>(end - start).count();
}

void runBenchmark(int size, const string& desc, auto dataGenFunc) {
    cout << "\n[데이터: " << desc << " | 배열 크기 N = " << size << "]\n";
    
    vector<int> baseData = dataGenFunc(size);
    
    // Bubble Sort (N=100000일 경우 너무 오래 걸리므로 제한)
    if (size <= 10000) {
        cout << " - Bubble Sort: " << fixed << setprecision(3) << measureTime(bubbleSort, baseData) << " ms\n";
    } else {
        cout << " - Bubble Sort: 너무 오래 걸림 (생략 혹은 별도 측정)\n";
    }
    
    // Quick Sort (역순 + 크기 100000일 경우 스택오버플로우 위험으로 제한)
    if (desc == "역순 (Reversed)" && size >= 100000) {
        cout << " - Quick Sort : 스택 오버플로우 방지로 실행 생략\n";
    } else {
        cout << " - Quick Sort : " << fixed << setprecision(3) << measureTime(quickSortWrapper, baseData) << " ms\n";
    }

    // Tim Sort
    cout << " - Tim Sort   : " << fixed << setprecision(3) << measureTime(timSort, baseData) << " ms\n";
}

int main() {
    cout << "========================================\n";
    cout << " 정렬 알고리즘 벤치마크 (Sorting Benchmark)\n";
    cout << "========================================\n";

    vector<int> sizes = {1000, 10000, 100000};

    for (int size : sizes) {
        runBenchmark(size, "무작위 (Random)", generateRandom);
        runBenchmark(size, "역순 (Reversed)", generateReversed);
        runBenchmark(size, "거의 정렬됨 (Nearly Sorted)", generateNearlySorted);
        cout << "----------------------------------------\n";
    }

    return 0;
}
