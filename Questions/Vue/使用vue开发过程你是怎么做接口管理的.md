# 使用 vue 开发过程你是怎么做接口管理的？

## 全局配置 api

在项目结构中新加一个 util 是的文件夹目录。在目录中创建一个 api.js
在 api 中声明一个或者多个 prefix 变量，用来切换不同的 host

根据接口所属的模块，声明多个模块对象，每个对象里多个属性，key 自己声明，value 是接口地址。

然后全局抛出所有的对象

在 Vue 的 main.js 里引入该 api 对象，然后绑定到 Vue 的原型链上，这样就可以在页面里使用的时候，直接使用 this.\$api.xxx 表示某个接口地址

如果在 action 里写的话，可以直接引入这个这个对象，在方法里使用 api.xx 表示接口

```js
// const prefix = '/mock/135/api';
const prefix = '/collector/api';

// 项目相关接口地址
const projectApi = {
  // 获取我参与的项目列表||全部项目列表
  queryProjects: `${prefix}/projects/myProjects`,
};
// 条目相关接口地址
const itemApi = {
  // 获取已编辑版本列表 || 获取草稿箱
  getItemList: `${prefix}/item/getItemList`,
  // 删除草稿箱
  deleteDrafts: `${prefix}/item/deleteItem`,
};

// 权限相关接口
const authorityApi = {
  // 获取人员列表
  getUsersList: `${prefix}/authority/getEditors`,
  // 获取项目邀请码
  getInviteCode: `${prefix}/authority/addWriters`,
};

// 用户相关接口
const userApi = {
  // 获取验证码
  getVerifyCode: `${prefix}/user/getVerifyCode`,
  // 登录接口
  userLogin: `${prefix}/user/login`,
};

const apis = {
  ...projectApi,
  ...itemApi,
  ...authorityApi,
  ...userApi,
};

export default apis;
```
