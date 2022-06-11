## v-if和v-show的区别

v-show的元素始终会被渲染并保存在DOM中，v-show只是简单的切换元素的CSS属性display。

而v-if是真正的条件渲染，因为它会确保切换过程中条件块内的时间监听和子组件适当地被销毁和重建。

## v-if和v-show的使用场合

一般而言，v-if有更高的切换开销，而v-show有更高的初始化渲染开销。因此如果要非常的频繁切换，则使用v-show。如果在运行的时候很少改变，则使用v-if较好。

## v-if、v-else、v-else-if用法

```
<div v-if="type === 'A'">
A
</div>
<div v-else-if="type === 'B'">
B
</div>
<div v-else-if="type === 'C'">
C
</div>
<div v-else>
Not A/B/C
</div>
```

## v-if 与 v-for 一起使用

当 v-if 与 v-for 一起使用时，v-for 具有比 v-if 更高的优先级，这意味着 v-if 将分别重复运行于每个 v-for 循环中

所以，不推荐v-if和v-for同时使用

使用推荐方式：

```
	<ul>
          <li
            v-for="(item, key) in activeItems"
            :key="key"
            :class="{ 'menu-item': true, current: current == item }">
            <router-link :to="item.path">{{item.title}}</router-link>
          </li>
        </ul>

  computed: {
    activeItems: function() {
      return this.navMenu.filter(function(item) {
        return !item.login || isLogin();
      });
    },
  },
```
