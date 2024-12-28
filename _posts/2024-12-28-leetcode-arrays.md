### Leetcode and it's worth
No one likes leetcode. At least not anyone that I know. Everyone considers it a necessary evil to get past FAANG and other tech interviews. I must have worked on Leetcode with the same attitude for nearly 3 years now. I am quite shocked by my own dashboard ![dashboard](https://i.imgur.com/N3yn7wH.png)
I had never thought that I would be able solve 300 problems on leetcode. It's only in the last year or so I realized that this number means nothing. If I randomly picked up one of the problems that I had solved and tried to solve again, I'd be stumped. 

Up until very recently I had trouble concentrating and getting to levels of deep work that I desired. Though this has improved in the last couple of months, I don't believe I am there yet. So let me walk through the saga of solving a medium level problem on Leetcode.

### The problem
I wanted to pick a problem at random and then this [Top 150 interview questions](https://leetcode.com/studyplan/top-interview-150/) list popped up. So I was like why not. Started crunching the problems and then came across this one. [Remove duplicates from sorted array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)

### The problem statement
Given an integer array nums sorted in non-decreasing order, remove some duplicates in-place such that each unique element appears at most twice. The relative order of the elements should be kept the same.

Since it is impossible to change the length of the array in some languages, you must instead have the result be placed in the first part of the array nums. More formally, if there are k elements after removing the duplicates, then the first k elements of nums should hold the final result. It does not matter what you leave beyond the first k elements.

Return k after placing the final result in the first k slots of nums.
Do not allocate extra space for another array. You must do this by modifying the input array in-place with O(1) extra memory.

### My approach
I thought how hard can this be. After all solving remove duplicates from sorted array was a breeze. So I started with the brute force approach.
`[0,0,1,1,1,1,2,3,3]`
Hashmap to store the count of each element was out since O(1) additional space is all I could use. So iterate through the array and mark the duplicate crossing the limit of 2. So my task was if I get to something like `[0,0,1,1,-1,-1,2,3,3]` then I am getting somewhere. Pehaps in a second pass I can move all the -1s to the end and return the count of non -1s.
But then to maintain a count of each occurance of the element I had to use a hashmap. No point. I was still hung up about the count and totally lost track of O(1) requirement. Spent nearly an hour on this and then I looked up the solution.

It was just 3 lines of code. Followed by a comment 'do we live in the same universe'. So I wasn't alone in the bewilderment.

### The solution
Having read through the solution, I quickly realized the charm of the solution. I am trying to solve for the problem by going through what is asked of me. The question is to remove duplicates. So I am fixated on the verb remove. The solution author instead does something very counter intuitive. 

```python
i = 0
for n in nums:
    if i < 2 or n != nums[i-2]:
```
The if condition completely threw me off. How could anybody arrive at such an intuition. `if i < 2` is obvious. But the second part is a complete mystery. When you are asked to remove the duplicates why is this guy checking the previous two elements. ok even if you generalize it is k elements so why `n != nums[i-k] or i < k`. The OR condition also threw me off.

Then I look at the OR condition a bit more closely and then it hits me, until i == 2 the condition will always be true. And then I see the next statement
```python
i = 0
for n in nums:
    if i < 2 or n != nums[i-2]:
        nums[i] = n
        i += 1
```
So until i == limit or until there are no consecutive duplicates, `nums[i] = n`. If both these conditions become true then nothing happens since it's OR. So you are not really removing the duplicates, you are efficiently rearranging the array without a whole lot of movement. That's how reframing the problem helps. You need to remove duplicates can be reformulated as don't remove the duplicates but replace them instead. Once you undertsand the replacement part, then the key comes when to replace. That's how I understood this solution. Not sure how long the solution will remain in my memory, but I am glad I understood it.

IMO understanding a problem by framing it in different ways is one of the key skills that algorithm tests check for. While I agree that real life is nothing like this but there are situations in which you need to look at problem statements from a different angle altogether to find even one solution, let alone the best solution.