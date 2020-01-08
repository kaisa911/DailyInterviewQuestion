# vue 中组件 name 的作用

```js
export default {
  name: 'xxx',
};
```

## 当项目使用 keep-alive 时，可搭配组件 name 进行缓存过滤

```js
export default {
  name:'User'
}，
mounted(){
  this.getUserInfo();
}，
methods:{
  getUserInfo(){
    axios.get('/xx/user.json',{
      params:{
        id:this.$route.params.id
      }
    }).then(this.getxxxInfo)
  }
}
```

如果在 App.vue 中使用了 keep-alive 导致我们第二次进入的时候页面不会重新请求，即触发 mounted 函数。
有两个解决方案.

- 增加 activated()函数,每次进入新页面的时候再获取一次数据。
- 在 keep-alive 中增加一个过滤，如下所示：

```js
<div id='app'>
  <keep-alive exclude='User'>
    <router-view />
  </keep-alive>
</div>
```

## DOM 做递归组件时

一个组件须在父组件中迭代调用自身时

```js
 <div>
        <div v-for="(item,index) of list" :key="index">
            <div>
                <span class="item-title-icon"></span>
                {{item.title}}
            </div>
            <div v-if="item.children" >
               <info-list :list="item.children"></info-list>
            </div>
        </div>
  </div>

<script>
export default {
    name:'InfolList',//组件自身调用自身
    props:{
        list:Array
    }
}
</script>

```

父组件中的数据样式

```js
const list = [
  {
    title: 'A',
    children: [
      {
        title: 'A-A',
        children: [
          {
            title: 'A-A-A',
          },
        ],
      },
      {
        title: 'A-B',
      },
    ],
  },
  {
    title: 'B',
  },
  {
    title: 'C',
  },
  {
    title: 'D',
  },
];
```

## vue-tools

vue-tools 中的组件名称解析的是 name
