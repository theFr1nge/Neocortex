---
title: Circularly Linked List
date: 2021-11-16T21:07:41+01:00
---
## Circularly Linked List
This is another type of specialized [[Linked List]]. Normally in a linked list, the tail node would have the *next* pointer set as null. However, in Circularly Linked Lists, we instead set the tail's next element to head, essentially **circling** back to the start of the list in the end. This approach eliminates the need for a seperate *head* variable in our implementation since *head* is simple `tail.getNext()`. This makes our code more memory-efficient and also makes our logic simpler, causing our code to be easier to read. A circular linked list looks like this:
![[Excalidraw/Drawing 2021-11-15 20.52.11.excalidraw.md]]