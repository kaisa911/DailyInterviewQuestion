# 组件中写 name 选项有什么作用？

我们在写 vue 项目的时候会给组件写 name 属性。这是一个非必须的属性。
先来看看文档里写的
![img](https://user-gold-cdn.xitu.io/2018/10/7/1664da9afb3f668a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

它大概包括三个功能：

## 递归组件运用（指组件自身组件调用自身组件）

```js
</template>
  <article>
    <div class="item" v-for="(item,index) in list" :key="index">
      <div class="item-title">
        <span class="item-title-ticket"></span>
        {{item.title}}</div>
        <div v-if="item.children" class="item-children">
        <detail-list :list="item.children"></detail-list>
      </div>
    </div>
  </article>
</template>
<script>
export default {
  name: "DetailList",  /*指组件自身组件调用自身组件*/
  props: {
    list: Array
  },
  data() {
    return {};
  }
};
</script>

```

## 当项目使用 keep-alive 时，可搭配组件 name 进行缓存过滤

```js
<div id='app'>
     {' '}
  <keep-alive exclude='compA'>
      <router-view /> 
  </keep-alive>
</div>

// exclude="compA"   keep-alive属性对compA组件不生效
```

## vue-tools 插件调试

- 组件未定义 name 属性
- 组件将显示成，这很没有语义。
- 通过提供 name 选项，可以获得更有语义信息的组件树。
