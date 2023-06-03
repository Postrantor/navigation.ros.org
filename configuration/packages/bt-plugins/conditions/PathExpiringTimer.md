---
title: PathExpiringTimer
---

Checks if the timer has expired. Returns success if the timer has expired, otherwise it returns failure. The timer will reset if the path gets updated.

# Example

``` xml
<PathExpiringTimer seconds="15" path="{path}"/>
```
