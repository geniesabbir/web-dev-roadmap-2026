# Day 3: Mastering Coding Interviews

## Introduction

Coding interviews are the most challenging part of the tech job search for many developers. They require a combination of problem-solving skills, data structure knowledge, and the ability to think out loud under pressure. Today, we'll cover everything you need to know to succeed in technical coding interviews.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Coding Interview Success Formula                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │                                                                 │  │
│   │    Problem       Communication     Technical        Practice    │  │
│   │    Solving    +     Skills      +  Knowledge    +             │  │
│   │    Approach                                                    │  │
│   │                                                                 │  │
│   │        │               │              │              │         │  │
│   │        ▼               ▼              ▼              ▼         │  │
│   │   Understand      Think out       DS & Algo      Consistent    │  │
│   │   → Clarify       loud clearly    fundamentals   daily prep    │  │
│   │   → Plan          → Ask Qs        → Big O        → Mock        │  │
│   │   → Code          → Explain       → Patterns     interviews    │  │
│   │   → Test          reasoning                                    │  │
│   │                                                                 │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

By the end of this lesson, you will:
- Understand the coding interview process and what interviewers look for
- Master the problem-solving framework (UMPIRE)
- Review essential data structures and algorithms
- Learn common problem patterns
- Practice with real interview problems

---

## 1. Understanding Coding Interviews

### What Interviewers Are Looking For

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Interview Evaluation Criteria                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PROBLEM SOLVING (40%)                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Can you break down complex problems?                          │   │
│  │  • Do you consider edge cases?                                   │   │
│  │  • Can you optimize your solution?                               │   │
│  │  • Do you handle ambiguity well?                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  CODING ABILITY (30%)                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Is your code clean and readable?                              │   │
│  │  • Do you use appropriate data structures?                       │   │
│  │  • Can you translate ideas into working code?                    │   │
│  │  • Do you handle errors properly?                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  COMMUNICATION (20%)                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Do you think out loud?                                        │   │
│  │  • Can you explain your approach clearly?                        │   │
│  │  • Do you ask clarifying questions?                              │   │
│  │  • Are you receptive to hints?                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  TECHNICAL KNOWLEDGE (10%)                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Do you know Big O notation?                                   │   │
│  │  • Can you analyze time/space complexity?                        │   │
│  │  • Do you understand tradeoffs?                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Interview Formats

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Common Interview Formats                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PHONE/VIDEO SCREEN (45-60 min)                                         │
│  • 1-2 coding problems                                                  │
│  • Usually on CoderPad, HackerRank, or similar                          │
│  • Focus: Can you code and communicate at a basic level?                │
│                                                                         │
│  ON-SITE/VIRTUAL ON-SITE (4-6 hours)                                    │
│  • Multiple rounds (typically 4-5)                                      │
│  • Mix of coding, system design, behavioral                             │
│  • May include lunch chat (not evaluated but matters!)                  │
│                                                                         │
│  TAKE-HOME ASSIGNMENT (2-8 hours)                                       │
│  • Build a small project or solve problems                              │
│  • Focus: Code quality, architecture, testing                           │
│  • Often followed by discussion of your solution                        │
│                                                                         │
│  LIVE CODING (PAIR PROGRAMMING)                                         │
│  • Work on a problem with the interviewer                               │
│  • More collaborative, hints available                                  │
│  • Focus: How you work with others                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. The UMPIRE Problem-Solving Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    UMPIRE Framework                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  U - UNDERSTAND THE PROBLEM (2-3 minutes)                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Read the problem carefully                                    │   │
│  │  • Identify inputs and outputs                                   │   │
│  │  • Ask clarifying questions:                                     │   │
│  │    - What's the input size/range?                                │   │
│  │    - Are there duplicates?                                       │   │
│  │    - Is the input sorted?                                        │   │
│  │    - What should I return for edge cases?                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  M - MATCH TO KNOWN PATTERNS (1-2 minutes)                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Does this remind me of a pattern I know?                      │   │
│  │  • Two pointers? Sliding window? BFS/DFS?                        │   │
│  │  • What data structure fits best?                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  P - PLAN YOUR APPROACH (3-5 minutes)                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Write out steps in plain English                              │   │
│  │  • Walk through with an example                                  │   │
│  │  • Get interviewer buy-in before coding                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  I - IMPLEMENT THE SOLUTION (15-20 minutes)                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Write clean, readable code                                    │   │
│  │  • Use meaningful variable names                                 │   │
│  │  • Keep talking through your code                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  R - REVIEW YOUR CODE (2-3 minutes)                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Walk through with the given example                           │   │
│  │  • Check edge cases                                              │   │
│  │  • Look for bugs (off-by-one, null checks)                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  E - EVALUATE COMPLEXITY (1-2 minutes)                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • State time complexity                                         │   │
│  │  • State space complexity                                        │   │
│  │  • Discuss potential optimizations                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### UMPIRE in Action: Example Problem

```javascript
// Problem: Two Sum
// Given an array of integers nums and an integer target,
// return indices of the two numbers that add up to target.

// U - UNDERSTAND
// Questions to ask:
// - Can I assume there's exactly one solution? (Yes)
// - Can I use the same element twice? (No)
// - Are there negative numbers? (Yes)
// - What's the size of the array? (Up to 10^4)

// M - MATCH
// This is a classic "find pair" problem
// Options: Brute force O(n²), Hash map O(n)

// P - PLAN
// 1. Create a hash map to store {value: index}
// 2. For each number, calculate complement = target - num
// 3. If complement exists in map, return [map[complement], i]
// 4. Otherwise, add current number to map
// 5. Return empty array if no solution (won't happen per problem)

// I - IMPLEMENT
function twoSum(nums, target) {
  const numMap = new Map();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (numMap.has(complement)) {
      return [numMap.get(complement), i];
    }

    numMap.set(nums[i], i);
  }

  return [];
}

// R - REVIEW
// Example: nums = [2, 7, 11, 15], target = 9
// i=0: complement=7, map={}, add 2:0 → map={2:0}
// i=1: complement=2, map has 2 → return [0, 1] ✓

// Edge cases:
// - [3, 3], target=6 → Works (second 3 finds first)
// - Negative numbers work with same logic

// E - EVALUATE
// Time: O(n) - single pass through array
// Space: O(n) - hash map stores up to n elements
```

---

## 3. Essential Data Structures

### Arrays and Strings

```javascript
// KEY OPERATIONS AND COMPLEXITIES
/*
┌─────────────────────────────────────────────────────────────────────────┐
│  Array Operations                    │  Time Complexity                │
├──────────────────────────────────────┼─────────────────────────────────┤
│  Access by index                     │  O(1)                           │
│  Search (unsorted)                   │  O(n)                           │
│  Search (sorted - binary)            │  O(log n)                       │
│  Insert at end                       │  O(1) amortized                 │
│  Insert at beginning                 │  O(n)                           │
│  Remove from end                     │  O(1)                           │
│  Remove from beginning               │  O(n)                           │
└──────────────────────────────────────┴─────────────────────────────────┘
*/

// COMMON PATTERNS

// 1. Two Pointers (opposite ends)
function isPalindrome(s) {
  let left = 0;
  let right = s.length - 1;

  while (left < right) {
    if (s[left] !== s[right]) return false;
    left++;
    right--;
  }
  return true;
}

// 2. Sliding Window
function maxSubarraySum(arr, k) {
  let maxSum = 0;
  let windowSum = 0;

  // First window
  for (let i = 0; i < k; i++) {
    windowSum += arr[i];
  }
  maxSum = windowSum;

  // Slide window
  for (let i = k; i < arr.length; i++) {
    windowSum = windowSum - arr[i - k] + arr[i];
    maxSum = Math.max(maxSum, windowSum);
  }

  return maxSum;
}

// 3. Prefix Sum
function subarraySum(nums, k) {
  const prefixCount = new Map([[0, 1]]);
  let sum = 0;
  let count = 0;

  for (const num of nums) {
    sum += num;
    if (prefixCount.has(sum - k)) {
      count += prefixCount.get(sum - k);
    }
    prefixCount.set(sum, (prefixCount.get(sum) || 0) + 1);
  }

  return count;
}
```

### Hash Maps and Sets

```javascript
// KEY OPERATIONS
/*
┌─────────────────────────────────────────────────────────────────────────┐
│  Hash Map/Set Operations             │  Time Complexity (average)      │
├──────────────────────────────────────┼─────────────────────────────────┤
│  Insert                              │  O(1)                           │
│  Delete                              │  O(1)                           │
│  Search                              │  O(1)                           │
│  Iterate                             │  O(n)                           │
└──────────────────────────────────────┴─────────────────────────────────┘
*/

// COMMON USE CASES

// 1. Frequency Counter
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const num of nums) {
    freq.set(num, (freq.get(num) || 0) + 1);
  }

  return [...freq.entries()]
    .sort((a, b) => b[1] - a[1])
    .slice(0, k)
    .map(([num]) => num);
}

// 2. Check for Duplicates
function containsDuplicate(nums) {
  const seen = new Set();
  for (const num of nums) {
    if (seen.has(num)) return true;
    seen.add(num);
  }
  return false;
}

// 3. Group Anagrams
function groupAnagrams(strs) {
  const groups = new Map();

  for (const str of strs) {
    const key = [...str].sort().join('');
    if (!groups.has(key)) {
      groups.set(key, []);
    }
    groups.get(key).push(str);
  }

  return [...groups.values()];
}
```

### Linked Lists

```javascript
// DEFINITION
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}

// KEY OPERATIONS
/*
┌─────────────────────────────────────────────────────────────────────────┐
│  Linked List Operations              │  Time Complexity                │
├──────────────────────────────────────┼─────────────────────────────────┤
│  Access by index                     │  O(n)                           │
│  Search                              │  O(n)                           │
│  Insert at head                      │  O(1)                           │
│  Insert at tail (with ref)           │  O(1)                           │
│  Insert at position                  │  O(n)                           │
│  Delete                              │  O(n) to find, O(1) to delete   │
└──────────────────────────────────────┴─────────────────────────────────┘
*/

// COMMON PATTERNS

// 1. Reverse Linked List
function reverseList(head) {
  let prev = null;
  let curr = head;

  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  return prev;
}

// 2. Fast and Slow Pointers (Find Middle)
function findMiddle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }

  return slow;
}

// 3. Detect Cycle
function hasCycle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }

  return false;
}

// 4. Merge Two Sorted Lists
function mergeTwoLists(l1, l2) {
  const dummy = new ListNode(0);
  let curr = dummy;

  while (l1 && l2) {
    if (l1.val <= l2.val) {
      curr.next = l1;
      l1 = l1.next;
    } else {
      curr.next = l2;
      l2 = l2.next;
    }
    curr = curr.next;
  }

  curr.next = l1 || l2;
  return dummy.next;
}
```

### Trees

```javascript
// DEFINITION
class TreeNode {
  constructor(val = 0, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

// TRAVERSALS

// 1. DFS - Preorder, Inorder, Postorder
function inorderTraversal(root) {
  const result = [];

  function dfs(node) {
    if (!node) return;
    dfs(node.left);        // Left first for inorder
    result.push(node.val); // Process node
    dfs(node.right);       // Right last
  }

  dfs(root);
  return result;
}

// 2. BFS - Level Order
function levelOrder(root) {
  if (!root) return [];

  const result = [];
  const queue = [root];

  while (queue.length) {
    const levelSize = queue.length;
    const level = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);

      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }

    result.push(level);
  }

  return result;
}

// COMMON PROBLEMS

// 1. Maximum Depth
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// 2. Validate BST
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;

  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}

// 3. Lowest Common Ancestor
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;

  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);

  if (left && right) return root;
  return left || right;
}
```

### Graphs

```javascript
// REPRESENTATIONS

// Adjacency List (most common)
const graph = {
  0: [1, 2],
  1: [0, 3],
  2: [0, 3],
  3: [1, 2],
};

// TRAVERSALS

// 1. BFS
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = [start];
  const result = [];

  while (queue.length) {
    const node = queue.shift();
    result.push(node);

    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }

  return result;
}

// 2. DFS
function dfs(graph, start) {
  const visited = new Set();
  const result = [];

  function explore(node) {
    if (visited.has(node)) return;
    visited.add(node);
    result.push(node);

    for (const neighbor of graph[node]) {
      explore(neighbor);
    }
  }

  explore(start);
  return result;
}

// COMMON PROBLEMS

// 1. Number of Islands (DFS on grid)
function numIslands(grid) {
  if (!grid.length) return 0;

  const rows = grid.length;
  const cols = grid[0].length;
  let count = 0;

  function dfs(r, c) {
    if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] === '0') {
      return;
    }

    grid[r][c] = '0'; // Mark visited
    dfs(r + 1, c);
    dfs(r - 1, c);
    dfs(r, c + 1);
    dfs(r, c - 1);
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === '1') {
        count++;
        dfs(r, c);
      }
    }
  }

  return count;
}

// 2. Course Schedule (Cycle Detection / Topological Sort)
function canFinish(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);

  for (const [course, prereq] of prerequisites) {
    graph[prereq].push(course);
    inDegree[course]++;
  }

  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.push(i);
  }

  let completed = 0;
  while (queue.length) {
    const course = queue.shift();
    completed++;

    for (const next of graph[course]) {
      inDegree[next]--;
      if (inDegree[next] === 0) queue.push(next);
    }
  }

  return completed === numCourses;
}
```

---

## 4. Common Algorithm Patterns

### Binary Search

```javascript
// Standard Binary Search
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return -1;
}

// Binary Search Variations

// Find first occurrence
function findFirst(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      result = mid;
      right = mid - 1; // Keep searching left
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return result;
}

// Binary Search on answer space
function minEatingSpeed(piles, h) {
  let left = 1;
  let right = Math.max(...piles);

  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    const hours = piles.reduce((sum, p) => sum + Math.ceil(p / mid), 0);

    if (hours <= h) {
      right = mid;
    } else {
      left = mid + 1;
    }
  }

  return left;
}
```

### Dynamic Programming

```javascript
// PATTERN 1: 1D DP

// Climbing Stairs
function climbStairs(n) {
  if (n <= 2) return n;

  let prev2 = 1;
  let prev1 = 2;

  for (let i = 3; i <= n; i++) {
    const curr = prev1 + prev2;
    prev2 = prev1;
    prev1 = curr;
  }

  return prev1;
}

// PATTERN 2: 2D DP

// Unique Paths
function uniquePaths(m, n) {
  const dp = Array(m).fill(null).map(() => Array(n).fill(1));

  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
  }

  return dp[m - 1][n - 1];
}

// PATTERN 3: String DP

// Longest Common Subsequence
function longestCommonSubsequence(text1, text2) {
  const m = text1.length;
  const n = text2.length;
  const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }

  return dp[m][n];
}

// PATTERN 4: Knapsack

// 0/1 Knapsack
function knapsack(weights, values, capacity) {
  const n = weights.length;
  const dp = Array(capacity + 1).fill(0);

  for (let i = 0; i < n; i++) {
    for (let w = capacity; w >= weights[i]; w--) {
      dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
    }
  }

  return dp[capacity];
}
```

### Backtracking

```javascript
// Template for backtracking
function backtrack(state, choices) {
  if (isComplete(state)) {
    result.push([...state]);
    return;
  }

  for (const choice of choices) {
    if (isValid(choice)) {
      state.push(choice);      // Make choice
      backtrack(state, ...);   // Explore
      state.pop();             // Undo choice
    }
  }
}

// EXAMPLE: Subsets
function subsets(nums) {
  const result = [];

  function backtrack(start, current) {
    result.push([...current]);

    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }

  backtrack(0, []);
  return result;
}

// EXAMPLE: Permutations
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);

  function backtrack(current) {
    if (current.length === nums.length) {
      result.push([...current]);
      return;
    }

    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;

      used[i] = true;
      current.push(nums[i]);
      backtrack(current);
      current.pop();
      used[i] = false;
    }
  }

  backtrack([]);
  return result;
}

// EXAMPLE: N-Queens
function solveNQueens(n) {
  const result = [];
  const board = Array(n).fill(null).map(() => Array(n).fill('.'));

  function isValid(row, col) {
    // Check column
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
    }
    // Check diagonals
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === 'Q') return false;
    }
    for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
      if (board[i][j] === 'Q') return false;
    }
    return true;
  }

  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }

    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = 'Q';
        backtrack(row + 1);
        board[row][col] = '.';
      }
    }
  }

  backtrack(0);
  return result;
}
```

---

## 5. Big O Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Big O Complexity Chart                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  COMPLEXITY      │  NAME           │  EXAMPLE                          │
│  ────────────────┼─────────────────┼───────────────────────────────────│
│  O(1)            │  Constant       │  Array access, hash lookup        │
│  O(log n)        │  Logarithmic    │  Binary search                    │
│  O(n)            │  Linear         │  Single loop, linear search       │
│  O(n log n)      │  Linearithmic   │  Merge sort, quick sort           │
│  O(n²)           │  Quadratic      │  Nested loops, bubble sort        │
│  O(2^n)          │  Exponential    │  Recursive fibonacci, subsets     │
│  O(n!)           │  Factorial      │  Permutations, traveling salesman │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PRACTICAL LIMITS (for ~1 second execution):                            │
│  ──────────────────────────────────────────                             │
│  O(n!)        →  n ≤ 10                                                 │
│  O(2^n)       →  n ≤ 20-25                                              │
│  O(n²)        →  n ≤ 3,000-5,000                                        │
│  O(n log n)   →  n ≤ 1,000,000                                          │
│  O(n)         →  n ≤ 10,000,000                                         │
│  O(log n)     →  n ≤ any reasonable size                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Practice Problems by Difficulty

### Easy (Warm Up)

```
1. Two Sum
2. Valid Palindrome
3. Merge Two Sorted Lists
4. Best Time to Buy and Sell Stock
5. Valid Parentheses
6. Maximum Subarray
7. Climbing Stairs
8. Reverse Linked List
9. Contains Duplicate
10. Binary Search
```

### Medium (Most Common in Interviews)

```
1. Add Two Numbers
2. Longest Substring Without Repeating Characters
3. Container With Most Water
4. 3Sum
5. Group Anagrams
6. Product of Array Except Self
7. Validate Binary Search Tree
8. Number of Islands
9. Course Schedule
10. Coin Change
11. Word Search
12. Top K Frequent Elements
13. Clone Graph
14. Pacific Atlantic Water Flow
15. Minimum Window Substring
```

### Hard (Stretch Goals)

```
1. Merge K Sorted Lists
2. Trapping Rain Water
3. Word Ladder
4. Serialize and Deserialize Binary Tree
5. Median of Two Sorted Arrays
6. Sliding Window Maximum
7. Longest Valid Parentheses
8. Regular Expression Matching
```

---

## 7. Interview Day Tips

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Interview Day Checklist                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  BEFORE THE INTERVIEW                                                   │
│  □ Test your audio/video setup                                          │
│  □ Have water nearby                                                    │
│  □ Close unnecessary applications                                       │
│  □ Review common patterns (not new problems)                            │
│  □ Prepare questions for your interviewer                               │
│                                                                         │
│  DURING THE INTERVIEW                                                   │
│  □ Take a deep breath before starting                                   │
│  □ Read the problem completely before coding                            │
│  □ Ask clarifying questions                                             │
│  □ Talk through your approach before coding                             │
│  □ Start with brute force if stuck, then optimize                       │
│  □ Test your code with examples                                         │
│  □ Mention time/space complexity                                        │
│                                                                         │
│  IF YOU GET STUCK                                                       │
│  □ Don't panic - it's normal                                            │
│  □ Talk through what you know                                           │
│  □ Ask for a hint (interviewers expect this)                            │
│  □ Try different examples                                               │
│  □ Consider different data structures                                   │
│                                                                         │
│  AFTER THE INTERVIEW                                                    │
│  □ Thank the interviewer                                                │
│  □ Send a follow-up thank you email                                     │
│  □ Review what went well and what to improve                            │
│  □ Don't dwell on mistakes - learn and move on                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Practice Strategy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    12-Week Study Plan                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  WEEKS 1-2: FOUNDATIONS                                                 │
│  • Review data structures (arrays, linked lists, trees, graphs)         │
│  • Practice 5-10 easy problems                                          │
│  • Focus on writing clean code                                          │
│                                                                         │
│  WEEKS 3-6: PATTERN MASTERY                                             │
│  • Learn each pattern with 3-5 problems                                 │
│  • Two pointers, sliding window, BFS/DFS                                │
│  • Binary search, dynamic programming, backtracking                     │
│  • 3-5 medium problems per day                                          │
│                                                                         │
│  WEEKS 7-10: PRACTICE AND SPEED                                         │
│  • Timed practice sessions (45 min per problem)                         │
│  • Mix of patterns                                                      │
│  • Focus on communication                                               │
│  • Start mock interviews                                                │
│                                                                         │
│  WEEKS 11-12: FINAL PREP                                                │
│  • Review weak areas                                                    │
│  • Do 2-3 mock interviews                                               │
│  • Light practice - don't burn out                                      │
│  • Focus on company-specific patterns                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Coding Interview Success                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Use the UMPIRE framework for every problem                          │
│  ✓ Communication matters as much as the solution                       │
│  ✓ Master patterns, not just individual problems                       │
│  ✓ Start with brute force, then optimize                               │
│  ✓ Practice consistently (quality > quantity)                          │
│  ✓ Do mock interviews to simulate pressure                             │
│  ✓ It's okay to ask for hints                                          │
│  ✓ Test your code before saying you're done                            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Additional Resources

- [LeetCode](https://leetcode.com/) - Main practice platform
- [NeetCode](https://neetcode.io/) - Organized problem lists by pattern
- [Blind 75](https://leetcode.com/discuss/general-discussion/460599/blind-75-leetcode-questions) - Must-do problems
- [AlgoExpert](https://www.algoexpert.io/) - Paid but excellent explanations
- [Tech Interview Handbook](https://www.techinterviewhandbook.org/)

---

Tomorrow, we'll tackle **System Design Interviews** - essential for senior roles and increasingly common at all levels.
