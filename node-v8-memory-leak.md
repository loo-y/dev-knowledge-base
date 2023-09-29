---
description: JavaScript heap out of memory
---

# Node v8 memory leak

node v8 内存泄漏

问题： Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory

解决：

```bash
#increase to 5gb
export NODE_OPTIONS="--max-old-space-size=5120" 

#increase to 6gb
export NODE_OPTIONS="--max-old-space-size=6144" 

#increase to 7gb
export NODE_OPTIONS="--max-old-space-size=7168" 

#increase to 8gb
export NODE_OPTIONS="--max-old-space-size=8192" 
```

