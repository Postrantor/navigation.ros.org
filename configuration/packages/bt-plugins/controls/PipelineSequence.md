---
title: PipelineSequence
---

Ticks the first child till it succeeds, then ticks the first and second children till the second one succeeds. It then ticks the first, second, and third children until the third succeeds, and so on, and so on. If at any time a child returns RUNNING, that doesn\'t change the behavior. If at any time a child returns FAILURE, that stops all children and returns FAILURE overall.

> 勾选第一个子项直到成功，然后勾选第一个和第二个子项直到第二个成功。然后勾选第一个、第二个和第三个子项，直到第三个成功，以此类推。**如果在任何时候一个子项返回`RUNNING`，这不会改变行为。如果在任何时候一个子项返回`FAILURE`，那么所有子项都会停止，并总体返回`FAILURE`**。

# Example

```xml
    <PipelineSequence>
        <!--Add tree components here--->
    </PipelineSequence>
```
