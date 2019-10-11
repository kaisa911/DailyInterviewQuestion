# vue-router 怎么配置 404 页面

前后端分离之后，路由拦截跳转的责任转移到了前端，后端只需返回接口数据。

## 思路

无论是单页还是多页，都是在前端的路由表中添加一个`path:'/404'`的路由，渲染相应的 404 页面。同时配置个规则，用户访问页面于路由表内所有页面都无法匹配时，便跳转到该页面。

## SPA 的 404 路由配置

### 路由表固定的情况

此时路由表是固定的，可以直接在路由表中添加个路径为'/404'的路由，同时在路由表底部配置一个路径为`*`的路由用于重定向，利用了路由表由上至下匹配的特点。

```js
// router.js
export default new Router({
  mode: 'history',
  routes: [
    // ...
    {
      name: '404',
      path: '/404',
      component: () => import('@/views/notFound.vue'),
    },
    {
      path: '*', // 此处需特别注意至于最底部
      redirect: '/404',
    },
  ],
});
```

### 路由表动态生成的情况

路由表是动态生成的情况下，一部分为基础路由表，一部分为根据用户信息动态生成的路由表。
若动态生成路由采用 addRouters 方法，将新路由规则在原路由表数组的尾部注入。这样应该使重定向规则在动态路由注入后再注入。
确保重定向规则在路由表最底部。

```js
// router.js
export default new Router({
  mode: 'history',
  routes: [
    // ...
    {
      name: '404',
      path: '/404',
      component: () => import('@/views/notFound.vue'),
    },
    // ...other codes
  ],
});
```

```js
// notFoundRouterMap.js

export default [
  {
    path: '*',
    redirect: '/404',
  },
];
```

```js
// main.js

//...other codes
router.beforeEach((to, from, next) => {
  new Promise((resolve, reject) => {
    if (getCookie(tokenName)) {
      if (!getInfo()) {
        Promise.all([store.dispatch('getBasicInfo'), store.dispatch('getUserDetail')]).then(res => {
          store.dispatch('GenerateRoutes', { roles }).then(() => {
          // 根据用户权限生成可访问的路由表
            router.addRoutes(store.getters.addRouters) // 动态添加可访问路由表
            router.addRoutes(NotFoundRouterMap) // 添加404及重定向路由规则
            resolve({ ...to, replace: true }) // 重新加载一次路由，让路由表更新成功后走下面else的判断
          })

        })
      } else {
        // ...other codes
      }
    } else {
      window.location.href = '/login.html'
    }
  }).then(res => {
    if (res) {
      next(res)
    } else {
      next()
    }
  }).catch(err => {
    new Error(err)
    next(false)
  })
```

## 多页应用的 404 路由配置

多页应用区别于单页在于每个页面拥有自己的一套路由，这时候就不能采用动态添加路由的方法。
此时可以利用全局路由守卫，beforeEach 中对每一个路由的匹配情况进行判断,这时候就需要用到 vue 导航守卫中的 matched 数组了。如果没有一个匹配上的跳转至 404 页面。

```js
// permission.js

//...other codes
router.beforeEach((to, from, next) => {
  new Promise((resolve, reject) => {
    // ...other codes
  }).then(res => {
    if (!to.matched.length) {
        window.location = '/error.html#/404'
        return
      }
    if (res) {
      next(res)
    } else {
      next()
    }
  }).catch(err => {
    new Error(err)
    next(false)
  })
```

这个方案就允许每个页面有自己的 404 页面路由规则，并且为没有配置 404 页面的路由统一配置了默认的 404 页面

## 参考

[](https://juejin.im/post/5b019ad7f265da0ba567d259)
