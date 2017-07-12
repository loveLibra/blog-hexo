title: Lodash源码祭(Date)
date: 2017-07-12 19:08:11
tags: [Lodash, Date]
---
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