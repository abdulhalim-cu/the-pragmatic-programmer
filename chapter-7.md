### **Chapter 7: While You Are Coding**

**Chapter Summary**

This chapter provides tactical, in-the-moment advice for the actual act of programming. It moves beyond high-level architecture and design to focus on the conscious, deliberate process of writing code line by line. A Pragmatic Programmer doesn't just type; they listen to their intuition, question their assumptions, and constantly strive for clarity and correctness. The chapter covers essential practices like refactoring, security awareness, and the critical importance of good naming, all unified by the theme of coding with intent and building quality in from the very first keystroke.

Here is a breakdown of the key topics covered in Chapter 7.

***

#### **Topic 37: Listen to Your Lizard Brain**

Your "lizard brain" refers to your intuition—that gut feeling that something is not quite right. It's the nagging doubt you feel when a piece of code is too complex, a task is taking too long, or a solution feels awkward. Pragmatic Programmers learn to trust this feeling. It's often a signal of a deeper problem, like a design flaw, a misunderstanding of the requirements, or a hidden "broken window." Don't ignore these feelings; investigate them.

*   **Core Idea:** Pay attention to your intuition. If something feels wrong, it probably is. Stop and analyze why you feel that way.

**Python Example (Heeding the Lizard Brain):**
Imagine you're writing a function and it's getting complicated with nested `if/else` statements.

```python
def calculate_shipping_cost(order, user):
    # LIZARD BRAIN ALERT: This nesting is getting too deep. It's hard to follow.
    # The feeling of "this is getting messy" is a signal to stop and rethink.
    if user.is_premium:
        cost = 0 # Premium users get free shipping
    else:
        if order.country == "US":
            if order.weight < 1.0:
                cost = 5.00
            else:
                cost = 10.00
        else: # International
            if order.weight < 1.0:
                cost = 15.00
            else:
                cost = 25.00
    return cost
```
Your "lizard brain" tells you this is fragile. A better approach might be to refactor it, perhaps using a data structure to store the rates, which is a much cleaner design.

***

#### **Topic 38: Programming by Coincidence**

This is the act of writing code that *seems* to work but for reasons you don't fully understand. You might be relying on a lucky side effect, an undocumented feature, or an implicit assumption about your environment. This is incredibly dangerous because a small, unrelated change can cause your code to break unexpectedly.

*   **Core Idea:** Code deliberately. Always know *why* your code works. Don't rely on luck or assumptions.
*   **Action:** If you're not sure how a library function behaves, write a small test to verify it. Read the documentation. Don't just copy-paste from a forum and hope for the best.

**Python Example (Relying on a Coincidence):**
Before Python 3.7, standard dictionaries did not preserve insertion order. Code that relied on this was programming by coincidence.
```python
# UNPRAGMATIC (Programming by Coincidence, especially in older Python versions)
steps = {"step1": "Extract", "step2": "Transform", "step3": "Load"}
# In Python < 3.7, iterating over this dict did not guarantee order.
# If it worked, it was a coincidence.

# PRAGMATIC (Deliberate Programming)
# The programmer knows WHY this works and chooses the right tool.
from collections import OrderedDict
ordered_steps = OrderedDict([("step1", "Extract"), ("step2", "Transform"), ("step3", "Load")])
# By using OrderedDict, we are explicitly stating that order is a requirement.
# This code is deliberate, not coincidental.
```

***

#### **Topic 39: Algorithm Speed**

It's crucial to have a rough idea of the performance characteristics (Big-O notation) of the algorithms you write and use. A solution that is fast for 10 items might be unacceptably slow for 10,000. You don't need a Ph.D. in computer science, but you should be able to spot common performance killers like nested loops or inefficient data structures.

*   **Core Idea:** Understand the complexity of your algorithms and choose them appropriately for the expected scale of your data.

**Python Example (O(n²) vs. O(n) ):**
Finding common elements between two lists.
```python
# UNPRAGMATIC (O(n*m) - Slow): Acceptable for small lists, disastrous for large ones.
def find_common_elements_slow(list1, list2):
    common = []
    for item1 in list1: # Nested loop
        for item2 in list2:
            if item1 == item2:
                common.append(item1)
    return common

# PRAGMATIC (O(n+m) - Fast): Scales much better.
def find_common_elements_fast(list1, list2):
    # Converting one list to a set makes lookups O(1) on average.
    set1 = set(list1)
    common = []
    for item2 in list2:
        if item2 in set1:
            common.append(item2)
    return common
# For two lists of 10,000 items each, the difference is seconds vs. microseconds.
```

***

#### **Topic 40: Refactoring**

Refactoring is the disciplined technique of restructuring existing code—changing the design—without changing its external behavior. It's not a scheduled task; it's a continuous, opportunistic activity. When you encounter code that is hard to understand, duplicated, or poorly designed (a "broken window"), you should improve it. Refactoring is like weeding a garden; do it often to prevent the weeds from taking over.

*   **Core Idea:** Continuously improve the design of the code you work with. Leave the code cleaner than you found it.

**Python Example (Refactoring a complex conditional):**
```python
# UNPRAGMATIC (Before Refactoring): A long, confusing function.
def calculate_price(order):
    if order.type == 'retail' and order.country == 'US':
        # ... complex retail logic for US ...
    elif order.type == 'wholesale' and order.country == 'US':
        # ... complex wholesale logic for US ...
    # ... more and more conditions ...

# PRAGMATIC (After Refactoring): Use a dispatch table (a dict).
def _calculate_retail_us(order):
    # ... logic ...
def _calculate_wholesale_us(order):
    # ... logic ...

PRICE_CALCULATORS = {
    ('retail', 'US'): _calculate_retail_us,
    ('wholesale', 'US'): _calculate_wholesale_us,
}

def calculate_price_refactored(order):
    key = (order.type, order.country)
    calculator = PRICE_CALCULATORS.get(key)
    if calculator:
        return calculator(order)
    else:
        # Default or error handling
        raise ValueError(f"No price calculator for {key}")
```

***

#### **Topic 41: Test to Code**

Writing tests *before* or *concurrently with* the code they test is a powerful way to think through your design. The tests act as the first user of your code, forcing you to create clean, usable interfaces. They also serve as an executable specification of what the code is supposed to do. This is the heart of Test-Driven Development (TDD).

*   **Core Idea:** Let your tests drive the development of your code. The cycle is: Red (write a failing test), Green (write the simplest code to make it pass), Refactor (clean up the code).

**Python Example (TDD for a simple `slugify` function):**
```python
import unittest

# Step 1: RED -> Write a failing test for a function that doesn't exist.
class TestSlugify(unittest.TestCase):
    def test_replaces_spaces_with_dashes(self):
        self.assertEqual(slugify("hello world"), "hello-world")

# Step 2: GREEN -> Write the simplest code to make the test pass.
def slugify(text):
    return text.replace(" ", "-")

# Step 3: REFACTOR/REPEAT -> Add a new test for another requirement.
# This test will fail.
# self.assertEqual(slugify("Hello World!"), "hello-world")

# Update the code to make the new test pass.
import re
def slugify(text):
    text = text.lower()
    text = re.sub(r'[^\w\s-]', '', text) # Remove special chars
    return re.sub(r'[-\s]+', '-', text).strip('-_')
# This cycle of test->code->refactor continues.
```

***

#### **Topic 42: Property-Based Testing**

While example-based testing (like TDD) checks for known cases, property-based testing checks for *general properties* or *invariants* that should always hold true. You define the property, and a testing framework (like `hypothesis` for Python) generates hundreds of random, valid inputs to try to find an edge case that falsifies it. This is excellent for finding bugs you never thought to test for.

*   **Core Idea:** Don't just test examples; test the fundamental properties of your code against a wide range of data.

**Python Example (Using `hypothesis`):**
The property: For any string `s`, decoding the result of `s.encode()` should give you back `s`.
```python
from hypothesis import given
import hypothesis.strategies as st

@given(st.text())
def test_decode_inverts_encode(s):
    # Hypothesis will generate all sorts of unicode strings to test this.
    # It will find edge cases you'd never think of, like empty strings,
    # weird unicode characters, etc.
    assert s.encode('utf-8').decode('utf-8') == s
```

***

#### **Topic 43: Stay Safe Out There**

This topic applies the "pragmatic paranoia" mindset directly to security while coding. You must assume that any data coming into your system from an external source is hostile. Never trust input from users, files, the network, or even other systems.

*   **Core Idea:** Assume all external input is an attack. Sanitize, validate, and escape everything.
*   **Common Pitfalls:** SQL injection, cross-site scripting (XSS), command injection, insecure deserialization.

**Python Example (Preventing Command Injection):**
```python
import subprocess

# UNPRAGMATIC (Vulnerable): Building a command string with user input.
def list_files_vulnerable(directory):
    # An attacker can provide a directory like: '.'; ls -la; echo 'pwned'
    command = f"ls -l {directory}"
    # Using shell=True with an un-sanitized string is extremely dangerous.
    subprocess.run(command, shell=True)

# PRAGMATIC (Safe): Passing command arguments as a list.
def list_files_safe(directory):
    # The arguments are passed separately. The OS will treat the malicious
    # string as a single, literal (and likely non-existent) directory name.
    # It is not interpreted as multiple commands.
    subprocess.run(['ls', '-l', directory])
```

***

#### **Topic 44: Naming Things**

"There are only two hard things in Computer Science: cache invalidation and naming things." This topic emphasizes the immense importance of good naming. Good names make code readable and self-documenting. Bad names (like single letters, abbreviations, or vague terms) create confusion, hide intent, and lead to bugs.

*   **Core Idea:** Choose names that are specific, descriptive, consistent, and reveal the code's purpose.

**Python Example (Poor vs. Good Naming):**
```python
# UNPRAGMATIC (Poor Naming): What does this code do?
def f(d_list):
    r = []
    for x in d_list:
        if x > 100:
            r.append(x)
    return r

# PRAGMATIC (Good Naming): The purpose is immediately clear.
def filter_high_value_transactions(transactions):
    high_value_transactions = []
    for transaction_amount in transactions:
        if transaction_amount > 100:
            high_value_transactions.append(transaction_amount)
    return high_value_transactions
```
