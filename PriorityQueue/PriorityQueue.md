### Implementation
In go, we change use heap to construct PriorityQueue, here's a example of building a small top heap. The smallest item alway on top.
```
import "container/heap"

type SmallHeap []int

func (h SmallHeap) Len() int {
    return len(h)
}

func (h SmallHeap) Less(i int, j int) bool {
    return (h)[i] < (h)[j]
}

func (h SmallHeap) Swap(i int, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *SmallHeap) Push(num any) {
    *h = append(*h, num.(int))
}

func (h *SmallHeap) Pop() any {
    x := (*h)[len(*h)-1]
    *h = (*h)[0 : len(*h)-1]
    return x
}

func main() {
    // Init
    h := &SmallHeap{}
    heap.Init(h)
    // Push
    heap.Push(h, 1)
    // Pop
    top := heap.Pop(h).(int)
}

```

### Examples
1. [leetcode 2336](https://leetcode.com/problems/smallest-number-in-infinite-set/description/)
```
type SmallestInfiniteSet struct {
    curr        int
    smallerHeap *SmallHeap
    heapMap     map[int]bool
}

func Constructor() SmallestInfiniteSet {
    h := &SmallHeap{}
    heap.Init(h)
    return SmallestInfiniteSet{0, h, map[int]bool{}}
}

func (this *SmallestInfiniteSet) PopSmallest() int {
    if this.smallerHeap.Len() > 0 {
        top := heap.Pop(this.smallerHeap).(int)
        this.heapMap[top] = false
        return top
    }
    this.curr++
    return this.curr
}

func (this *SmallestInfiniteSet) AddBack(num int) {
    if num <= this.curr && !this.heapMap[num] {
        this.heapMap[num] = true
        heap.Push(this.smallerHeap, num)
    }
}
```

2. [leetcode 2542](https://leetcode.com/problems/maximum-subsequence-score/description/)
```
func MaxScore(nums1 []int, nums2 []int, k int) int64 {
    n := len(nums1)
    h := SmallHeap{}
    heap.Init(&h)

    var pairs [][2]int
    for i := 0; i < n; i++ {
        pairs = append(pairs, [2]int{nums2[i], nums1[i]})
    }

    sort.Slice(pairs, func(i, j int) bool {
        return pairs[i][0] > pairs[j][0]
    })

    minVal, sum, ans := 0, 0, int64(0)
    for i := 0; i < n; i++ {
        minVal = pairs[i][0]
        sum += pairs[i][1]
        heap.Push(&h, pairs[i][1])

        if h.Len() > k {
            top := heap.Pop(&h).(int)
            sum -= top
        }

        if h.Len() == k {
            result := int64(minVal * sum)
            if result > ans {
                ans = result
            }
        }
    }
    return ans
}
```

3. [leetcode 2462](https://leetcode.com/problems/total-cost-to-hire-k-workers/description/)
```
func totalCost(costs []int, k int, candidates int) int64 {
    n := len(costs)
    lHeap := &SmallHeap{}
    rHeap := &SmallHeap{}
    heap.Init(lHeap)
    heap.Init(rHeap)
    var ans int64
    l, r := 0, n-1

    // fill heap
    for i := 0; i < candidates; i++ {
        if l <= r {
            heap.Push(lHeap, costs[l])
            l++
        }
        if l <= r {
            heap.Push(rHeap, costs[r])
            r--
        }
    }

    for i := 0; i < k; i++ {
        if lHeap.Len() > 0 && rHeap.Len() > 0 {
            if (*lHeap)[0] <= (*rHeap)[0] {
                ans += int64(heap.Pop(lHeap).(int))
                if l <= r {
                    heap.Push(lHeap, costs[l])
                    l++
                }
            } else {
                ans += int64(heap.Pop(rHeap).(int))
                if l <= r {
                    heap.Push(rHeap, costs[r])
                    r--
                }
            }
        } else if lHeap.Len() > 0 {
            ans += int64(heap.Pop(lHeap).(int))
        } else {
            ans += int64(heap.Pop(rHeap).(int))
        }
    }
    return ans
}
```