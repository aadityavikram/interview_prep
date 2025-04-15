# Intuition
- Traverse on nums2
- For every index, count how many numbers are present to the left of it in both arrays
- For every index, count how many numbers are present to the right of it in both arrays
- Answer will be the sum(product of both)

# Approach
- Linear Search
    - Store index of all elements on nums1 in an array
    - Traverse over nums2
    - Get index of current element of nums2 in nums1
    - Get index of all elements to the left of it in nums2 and find their index in nums1
    - If their index in nums1 also lie to the left of the current index, increase left count
    - Similarly count for the right side of current element
    - ans += left*right
- Binary Search
    - Store index of all elements on nums1 in an array
    - Instead of traversing over nums2 and finding index of its elements in nums1 one by one, store it in an array
    - Suppose idxNums2 is array that contains index of nums2's elements in nums1
    - Now the problem is reduced to finding, idxNums2[i], idxNums2[j], idxNums2[k] such that i<j<k
    - Perform binary search to find count of idxNums2[i] for every j
    - Perform binary search to find count of idxNums2[k] for every j
    - left[i] = count(idxNums2[i])
    - right[i] = count(idxNums2[k])
    - ans += left[i]*right[i]

# Complexity
- Time complexity:
    - Linear Search: O(N^2)
    - Binary SearchL O(N^2)

- Space complexity:
    - Linear Search: O(N)
    - Binary Search: O(N)

# Code with Linear Search | TLE
```java []
class Solution {
    public long goodTriplets(int[] nums1, int[] nums2) {
        long res = 0;

        // store index of every element in nums1
        int[] indexNums1 = new int[nums1.length];
        for (int i=0; i<nums1.length; i++) {
            indexNums1[nums1[i]] = i;
        }

        for (int i=0; i<nums2.length; i++) {
            int ele = nums2[i];
            int idx = indexNums1[ele];

            // count how many elements on left of current element in nums2
            // are left of current element in nums1
            // for ex. nums1: [4,0,1,3,2], nums2: [4,1,0,2,3]
            // we have to count elements on left of 0 in nums2
            // which are also on its left in nums1
            // elements on left of 0 in both: [4]
            // left_0 = 1
            int left = 0;
            for (int j=0; j<i; j++) {
                int ele2 = nums2[j];
                int idx2 = indexNums1[ele2];
                if (idx2 < idx) {
                    left += 1;
                }
            }

            // count how many elements on right of current element in nums2
            // are right of current element in nums1
            // similarly elements on right of 0 in both: [2,3]
            // right_0 = 2
            int right = 0;
            for (int j=i+1; j<nums2.length; j++) {
                int ele2 = nums2[j];
                int idx2 = indexNums1[ele2];
                if (idx2 > idx) {
                    right += 1;
                }
            }

            res += (long) left*right;
        }
        
        return res;
    }
}
```

# Code with Binary Search
```java []
class Solution {
    private int binSearch(List<Integer> nums, int target) {
        int low = 0;
        int high = nums.size()-1;
        while (low <= high) {
            int mid = low + (high-low)/2;
            if (nums.get(mid) == target) {
                return mid;
            }
            if (nums.get(mid) > target) {
                high = mid-1;
            } else {
                low = mid + 1;
            }
        }
        return low;
    }

    public long goodTriplets(int[] nums1, int[] nums2) {
        long res = 0;

        // idxNums1 stores index of nums1's elements
        // for ex. nums1: [4,0,1,3,2]
        // idxNums1: [1,2,4,3,0]
        int[] idxNums1 = new int[nums1.length];
        for (int i=0; i<nums1.length; i++) {
            idxNums1[nums1[i]] = i;
        }

        // idxNums2 stores index of nums2's elements in nums1
        // for ex. nums1: [4,0,1,3,2], nums2: [4,1,0,2,3]
        // idxNums2[0] = index of 4 in nums1 = 0
        // idxNums2: [0,2,1,4,3]
        // calculating idxNums2 will change this question to
        // finding count of idxNums2[i], idxNums2[j], idxNums2[k]
        // such that i<j<k
        int[] idxNums2 = new int[nums2.length];
        for (int i=0; i<nums2.length; i++) {
            idxNums2[i] = idxNums1[nums2[i]];
        }

        List<Integer> found = new ArrayList<>();

        // left[i] will store count of elements less than idxNums2[i] from 0 to i-1
        // right[i] will store count of elements greater than idxNums2[i] from i+1 to n-1
        // binary search is used to find the counts
        int[] left = new int[nums1.length];
        int[] right = new int[nums2.length];

        for (int i=0; i<nums1.length; i++) {
            int idx = idxNums2[i];
            // binary search will tell appropriate place to save idx in sorted order
            // all elements before idx will be lesser because found list is sorted
            int pos = binSearch(found, idx);
            left[i] = pos;
            found.add(pos, idx);
        }

        found = new ArrayList<>();
        for (int i=nums1.length-1; i>=0; i--) {
            int idx = idxNums2[i];
            // binary search will tell appropriate place to save idx in sorted order
            // all elements after idx will be greater because found list is sorted
            int pos = binSearch(found, idx);
            right[i] = found.size() - pos;
            found.add(pos, idx);
        }

        for (int i=0; i<nums1.length; i++) {
            res += (long) left[i] * right[i];
        }
        
        return res;
    }
}
```
