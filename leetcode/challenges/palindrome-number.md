---
description: https://leetcode.com/problems/palindrome-number
---

# Palindrome Number

Given an integer `x`, return `true` if `x` is palindrome integer.

An integer is a **palindrome** when it reads the same backward as forward.

* For example, `121` is a palindrome while `123` is not.

{% code title="solution.py" %}
```python
class Solution(object):
    def isPalindrome(self, x):
        """
        :type x: int
        :rtype: bool
        """
        temp = str(x)
        temp2 = str(x)
        temp2 = temp2[::-1]
        if temp == temp2:
            return True
        else:
            return False
```
{% endcode %}
