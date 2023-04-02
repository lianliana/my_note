# React学习心得



## State 如何设计

- 可以先把需要的都设置为State
- 接着把可以通过props / state推导的去掉
- 考虑如果 state 经常一起更新的 可以合并 如果不会同时true的 可以设置为string 如 typing submitting
- 还可以考虑合并一些state 如果成为了对象或者数组 记得考虑要浅拷贝的问题(可以通过useImmer去解决)



##  preserving-and-resetting-state 什么时候会更新/什么时候会保留状态



- 位置相同的时候 例如使用