title: Lodash源码祭「Date」
date: 2017-07-12 19:08:11
tags: [Lodash, Date]
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

# Date

## now
```javascript
_.now()
```
获取当前时间的毫秒时间戳
```javascript
const now () => {
    return new Date().getTime();
}
```