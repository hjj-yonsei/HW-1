# HW-1
import time
import random
import sys

# 파이썬의 재귀 한도 해제 (병합정렬용)
sys.setrecursionlimit(10**6)

# 1. 버블 정렬 (Bubble Sort)
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break
    return arr

# 2. 병합 정렬 (Merge Sort)
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    merged = []
    l = h = 0
    while l < len(left) and h < len(right):
        if left[l] < right[h]:
            merged.append(left[l])
            l += 1
        else:
            merged.append(right[h])
            h += 1
    merged += left[l:]
    merged += right[h:]
    return merged

# 3. 힙 정렬 (Heap Sort) - AI를 통해 학습한 정렬
def heapify(arr, n, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    # 왼쪽 자식이 루트보다 크면 largest 갱신
    if left < n and arr[i] < arr[left]:
        largest = left

    # 오른쪽 자식이 현재 largest보다 크면 갱신
    if right < n and arr[largest] < arr[right]:
        largest = right

    # largest가 루트가 아니라면 교환하고 다시 heapify
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

def heap_sort(arr):
    n = len(arr)
    
    # 최대 힙(Max Heap) 구성
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)

    # 요소를 하나씩 추출하여 정렬
    for i in range(n - 1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]  # 가장 큰 요소(루트)를 맨 뒤로
        heapify(arr, i, 0)               # 줄어든 힙에 대해 다시 heapify 적용
    return arr

# === 성능 측정 테스트 ===
if __name__ == "__main__":
    sizes = [1000, 5000, 10000]
    
    for size in sizes:
        print(f"\n[데이터 크기: {size}]")
        original_data = [random.randint(1, 100000) for _ in range(size)]
        
        # 1. Bubble Sort 테스트
        data = original_data.copy()
        start = time.time()
        bubble_sort(data)
        end = time.time()
        print(f"Bubble Sort : {end - start:.4f} 초")
        
        # 2. Merge Sort 테스트
        data = original_data.copy()
        start = time.time()
        merge_sort(data)
        end = time.time()
        print(f"Merge Sort  : {end - start:.4f} 초")
        
        # 3. Heap Sort 테스트
        data = original_data.copy()
        start = time.time()
        heap_sort(data)
        end = time.time()
        print(f"Heap Sort   : {end - start:.4f} 초")
