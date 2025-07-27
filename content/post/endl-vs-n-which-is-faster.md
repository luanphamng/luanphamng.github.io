+++
author = "Luan Pham"
title = "\"std::endl\" vs \"\\n\" which is faster"
date = "2024-08-19"
description = "\"std::endl\" vs \"\n\" which is faster."
tags = [
    "C++",
]
thumbnail = "images/post_thump/endl-vs-newline.jpg"

+++


Today, we will discuss basic techniques in C++. It is the new line character.
In C++ we have two ways to print out a new lines.

1. printf("\n");
2. using std::cout << std::endl;

The question is which one is faster?
__Answer__:
```'\n'``` is significantly faster than ```std::endl``` in C++.
Here's why:
```\n``` is simply a newline character. It inserts a new line into the output stream without any additional operations.
```std::endl``` does two things:
1. Inserts a newline character.
2. Flushes the output buffer.

Flushing the output buffer forces the contents of the buffer to be written to the actual output device (like the console or a file). This operation can be quite expensive, especially if done frequently.
In most cases, you don't need to explicitly flush the output buffer. The buffer is automatically flushed when:
+ The buffer is full.
+ The program terminates.
+ An input operation is performed (e.g., cin).

Therefore, using ```\n``` is generally the preferred choice for performance reasons. Only use ```std::endl``` if you specifically need to force a flush of the output buffer.

**Example**:
```C++
#include 
int main() {
    for (int i = 0; i < 1000000; ++i) {
        std::cout << "Hello, world!\n"; // Faster
        // std::cout << "Hello, world!" << std::endl; // Slower
    }
    return 0;
}
```
Use code with caution.
In this example, using ```\n``` will result in significantly faster execution compared to using ```std::endl```.
Remember: While performance is important, readability and maintainability are also crucial factors. If you need to explicitly flush the output buffer for specific reasons, using ```std::endl``` is acceptable. However, in most cases, \n is the better choice.

Would you like to know more about output buffering or other performance optimization techniques?
Follow me on social.

