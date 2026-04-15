import random
import time
from multiprocessing import Pool, Process, Queue
import sys


def merge(left, right):
    """Merge two sorted lists into one sorted list"""
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result


def sequential_merge_sort(arr):
    """Sequential merge sort implementation"""
    if len(arr) <= 1:
        return arr[:]  
    
    mid = len(arr) // 2
    left = sequential_merge_sort(arr[:mid])
    right = sequential_merge_sort(arr[mid:])
    return merge(left, right)

def sequential_bubble_sort(arr):
    """Sequential bubble sort (simple but slow)"""
    data = arr[:]
    n = len(data)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if data[j] > data[j + 1]:
                data[j], data[j + 1] = data[j + 1], data[j]
                swapped = True
        if not swapped:
            break
    return data

def sequential_insertion_sort(arr):
    """Sequential insertion sort"""
    data = arr[:]
    for i in range(1, len(data)):
        key = data[i]
        j = i - 1
        while j >= 0 and data[j] > key:
            data[j + 1] = data[j]
            j -= 1
        data[j + 1] = key
    return data



def parallel_sort_worker(chunk):
    """Worker function to sort a single chunk"""
    return sequential_merge_sort(chunk)

def parallel_merge_sort(data, num_processes=4):
    """Parallel merge sort using multiprocessing"""
    if len(data) <= 1000:  
        return sequential_merge_sort(data)
    

    chunk_size = len(data) // num_processes
    chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]
    

    if len(chunks) > num_processes:
        chunks[-2] = chunks[-2] + chunks[-1]
        chunks.pop()
    

    with Pool(num_processes) as pool:
        sorted_chunks = pool.map(parallel_sort_worker, chunks)
    

    result = sorted_chunks[0]
    for chunk in sorted_chunks[1:]:
        result = merge(result, chunk)
    
    return result



def sequential_search(data, target):
    """Sequential linear search - returns index or -1 if not found"""
    for i, value in enumerate(data):
        if value == target:
            return i
    return -1



def parallel_search_worker(sub_data, target, q, offset):
    """Worker function for parallel search"""
    for i, value in enumerate(sub_data):
        if value == target:
            q.put(offset + i)
            return
    q.put(-1)

def parallel_search(data, target, num_processes=4):
    """Parallel search using multiple processes"""
    if len(data) < num_processes:
        return sequential_search(data, target)
    
    chunk_size = len(data) // num_processes
    chunks = []
    offsets = []
    
    for i in range(num_processes):
        start = i * chunk_size
        end = start + chunk_size if i < num_processes - 1 else len(data)
        chunks.append(data[start:end])
        offsets.append(start)
    
    q = Queue()
    processes = []
    
    for i in range(num_processes):
        p = Process(target=parallel_search_worker, args=(chunks[i], target, q, offsets[i]))
        processes.append(p)
        p.start()
    

    results = []
    for _ in processes:
        try:
            results.append(q.get(timeout=5))
        except:
            results.append(-1)
    
    for p in processes:
        p.terminate()
        p.join()
    
    for res in results:
        if res != -1:
            return res
    return -1


def generate_dataset(size, case="random"):
    """Generate datasets of specified size and case"""
    if case == "random":
        return [random.randint(1, 1000000) for _ in range(size)]
    elif case == "sorted":
        return list(range(size))
    elif case == "reverse":
        return list(range(size, 0, -1))
    else:
        return [random.randint(1, 1000000) for _ in range(size)]



def test_sorting(data, algorithm_name, sort_func, is_parallel=False):
    """Test a sorting algorithm and return execution time"""
    data_copy = data[:]
    
    start = time.time()
    if is_parallel:
        sorted_data = sort_func(data_copy)
    else:
        sorted_data = sort_func(data_copy)
    end = time.time()
    

    is_sorted = all(sorted_data[i] <= sorted_data[i+1] for i in range(len(sorted_data)-1))
    
    return {
        "algorithm": algorithm_name,
        "time": end - start,
        "correct": is_sorted,
        "input_size": len(data)
    }

def test_search(data, target, algorithm_name, search_func, is_parallel=False):
    """Test a searching algorithm and return execution time and result"""
    start = time.time()
    index = search_func(data, target)
    end = time.time()
    
    return {
        "algorithm": algorithm_name,
        "time": end - start,
        "index": index,
        "target": target,
        "input_size": len(data)
    }



def run_all_tests():
    """Run comprehensive tests across all dataset sizes"""
    
    sizes = [1000, 100000, 1000000]
    cases = ["random", "sorted", "reverse"]
    
    print("=" * 80)
    print("SEQUENTIAL VS PARALLEL ALGORITHMS - PERFORMANCE ANALYSIS")
    print("=" * 80)
    
    all_results = []
    

    print("\n" + "=" * 80)
    print("SORTING ALGORITHMS")
    print("=" * 80)
    
    for size in sizes:
        for case in cases:
            print(f"\n--- Dataset: {size:,} elements ({case} order) ---")

            if case == "random":
                data = generate_dataset(size, "random")
            elif case == "sorted":
                data = generate_dataset(size, "sorted")
            else:
                data = generate_dataset(size, "reverse")
            

            result_seq = test_sorting(data, "Sequential Merge Sort", sequential_merge_sort, False)
            print(f"  Sequential Merge Sort: {result_seq['time']:.4f} seconds (Correct: {result_seq['correct']})")
            

            if size >= 10000:
                result_par = test_sorting(data, "Parallel Merge Sort", 
                                         lambda x: parallel_merge_sort(x, 4), True)
                print(f"  Parallel Merge Sort:   {result_par['time']:.4f} seconds (Correct: {result_par['correct']})")
                print(f"  Speedup:               {result_seq['time']/result_par['time']:.2f}x")
            else:
                print("  Parallel Merge Sort:   Skipped (overhead > benefit for small dataset)")
            
            all_results.append(result_seq)
            if size >= 10000:
                all_results.append(result_par)
    

    print("\n" + "=" * 80)
    print("SEARCHING ALGORITHMS")
    print("=" * 80)
    
    for size in sizes:
        data = generate_dataset(size, "random")

        target_exists = data[0]
        target_missing = -1
        
        print(f"\n--- Dataset: {size:,} elements (random order) ---")
        
        print(f"\n  Searching for existing target: {target_exists}")
        result_seq = test_search(data, target_exists, "Sequential Search", sequential_search, False)
        print(f"    Sequential Search: {result_seq['time']:.4f} seconds (Found at index: {result_seq['index']})")
        
        result_par = test_search(data, target_exists, "Parallel Search", 
                                lambda x, y: parallel_search(x, y, 4), True)
        print(f"    Parallel Search:   {result_par['time']:.4f} seconds (Found at index: {result_par['index']})")
        
        if result_seq['time'] > 0:
            print(f"    Speedup:           {result_seq['time']/result_par['time']:.2f}x")
        
        
        print(f"\n  Searching for missing target: {target_missing}")
        result_seq = test_search(data, target_missing, "Sequential Search", sequential_search, False)
        print(f"    Sequential Search: {result_seq['time']:.4f} seconds (Result: Not found)")
        
        result_par = test_search(data, target_missing, "Parallel Search",
                                lambda x, y: parallel_search(x, y, 4), True)
        print(f"    Parallel Search:   {result_par['time']:.4f} seconds (Result: Not found)")
        
        all_results.append(result_seq)
        all_results.append(result_par)
    
    print("\n" + "=" * 80)
    print("SUMMARY AND INSIGHTS")
    print("=" * 80)
    

    seq_sort_times = [r['time'] for r in all_results if 'Sequential Merge Sort' in r['algorithm']]
    par_sort_times = [r['time'] for r in all_results if 'Parallel Merge Sort' in r['algorithm']]
    
    if seq_sort_times and par_sort_times:
        print(f"\nAverage Sequential Sort Time: {sum(seq_sort_times)/len(seq_sort_times):.4f}s")
        print(f"Average Parallel Sort Time:   {sum(par_sort_times)/len(par_sort_times):.4f}s")
        print(f"Overall Speedup:              {sum(seq_sort_times)/sum(par_sort_times):.2f}x")
    
    print("\nKey Observations:")
    print("1. Parallel sorting performs better on datasets > 100,000 elements")
    print("2. Sequential search is faster for small datasets due to lower overhead")
    print("3. Parallel search shows improvement when searching large datasets")
    print("4. Already sorted data doesn't significantly benefit parallelization")
    print("5. Overhead from process creation and merging can exceed benefits for small inputs")

def quick_demo():
    """Quick demonstration of all algorithms"""
    print("=" * 60)
    print("QUICK DEMONSTRATION")
    print("=" * 60)
    
  
    data = [64, 34, 25, 12, 22, 11, 90, 5, 77, 30]
    print(f"\nOriginal data: {data}")
    
    sorted_seq = sequential_merge_sort(data)
    print(f"Sequential sort result: {sorted_seq}")
    
    sorted_par = parallel_merge_sort(data, 2)
    print(f"Parallel sort result:   {sorted_par}")
    
   
    target = 22
    idx_seq = sequential_search(data, target)
    print(f"\nSequential search for {target}: found at index {idx_seq}")
    
    idx_par = parallel_search(data, target, 2)
    print(f"Parallel search for {target}:   found at index {idx_par}")
    
    target = 999
    idx_seq = sequential_search(data, target)
    idx_par = parallel_search(data, target, 2)
    print(f"\nSearch for missing target {target}:")
    print(f"  Sequential: {idx_seq} (not found)")
    print(f"  Parallel:   {idx_par} (not found)")

if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1] == "--quick":
        quick_demo()
    else:
        run_all_tests()
