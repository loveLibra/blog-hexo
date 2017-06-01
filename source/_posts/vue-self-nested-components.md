title: vue组件自调用问题
date: 2017-06-01 22:37:35
tags: ['Vue', 'Solution']
---

## background
vuex单一数据中心后，中转动态组件，嵌套的多层结构的组件的渲染的解决方案

```javascript
// cross.vue
<component :is="type"></component>
```

```javascript
// component1.vue
<template>
    <div class="components-1">
        <cross :data="data"></cross>
    </div>
</template>

<script>
    import Corss from './cross.vue';
</script>
```

期望达到预期:
```html
<div class="components-1">
    ...
    <div class="components-1">
    ...
    </div>
</div>
```

然后发现在第一层的时候，import进来的Cross打印是组件本身内容，但是嵌套组件中再调用就不对了，打印为`{}` ...

## solution

[solution by stackoverflow](https://stackoverflow.com/questions/42634488/nested-components-in-vue-js-failed-to-mount-component-template-or-render-funct)

在`beforeCreate`中注册corss组件
```javascript
beforeCreate() {
    this.$options.components.cross = require('./cross.vue');
}
```
就可以了...


Why?

