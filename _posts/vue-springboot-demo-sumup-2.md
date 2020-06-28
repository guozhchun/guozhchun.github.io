---
title: 一个微型项目总结
date: 2019-06-09 20:06:28
tags: [springboot, vue, mybatis]
categories: 其他
---

# 前言

经过几个周末的学习和编码，终于把「微型项目：机器管理系统」给搭建起来了。本文是对本次开发过程中遇到的问题以及一些知识点的总结。

<!-- more -->

# 简介

首先，开发这个系统的目的主要是记录部门机器的使用情况，并提供公示（查询）的能力，方便机器管理，提高机器有效使用率。所以，项目一开始就本着快速搭建的初衷进行，这也使得在一些功能的实现上选择了简单轻便的方式进行处理，而这也会导致一些非功能性需求的缺失（如：性能、安全）。

## 系统功能

* 普通用户（非登录用户）可以对机器进行查询，包括按一定条件搜索机器
* 管理员（登录用户，系统有且只有一个账号）可以对机器（宿主机和虚拟机）进行管理，包括：增加、编辑、删除
* 支持中英文两种语言切换

## 系统截图

* 普通用户中文界面

![普通用户中文界面](/images/machine-management-4.png)

* 普通用户英文界面

![普通用户英文界面](/images/machine-management-5.png)

* 登录界面

![登录界面](/images/machine-management-6.png)

* 管理员界面

![管理员界面](/images/machine-management-7.png)

* 增加机器界面

![增加机器界面](/images/machine-management-2.png)

* 增加虚拟机界面

![增加虚拟机j界面](/images/machine-management-3.png)

# 知识点及问题

## vue 组件

这个主要是用在登录模块和条件查询模块。刚开始以为这是一个很小的系统，代码量应该很少，可以直接放在一个文件中。后面随着功能的增加和完善，代码越来越多，如果全部放入一个文件，将显得臃肿难看。所以就学习应用 vue 组件的思想和功能，将比较独立的「登录」和「条件查询」这两个功能抽成「组件」进行使用。

vue 组件有三个主要点：定义组件、注册组件、使用组件、组件通信

* 定义组件

  组件定义跟 vue 实例的定义类似，不同的是相关的 html 代码写在 `template` 元素中，data 元素是一个函数，这样才能保证各个组件之间不会相互影响。以下代码是条件查询的组件定义相关代码。

  ```vue
  var queryBlock = {
      props: ["machineKinds"],
      data: function () {
          return {
              queryCondition: {}
          }
      },
      methods: {
          query() {
              this.queryCondition.startPageNum = 1;
              this.queryCondition.pagePerCount = 10;
              this.$emit("get-data", this.queryCondition);
          }
      },
      template: `
          <el-collapse accordion>
              <el-collapse-item :title="$t('message.query')">
                  <el-form :model="queryCondition" label-position="right" label-width="120px">
                      <el-row :gutter="20">
                          <el-col :span="12" >
                              <el-form-item :label="$t('message.serialNumber')">
                                  <el-input clearable v-model="queryCondition.serialNumber" :placeholder="$t('message.tips.input.serialNumber')"></el-input>
                              </el-form-item>
                              <el-form-item :label="$t('message.type')" required>
                                  <el-select v-model="queryCondition.kind" clearable :placeholder="$t('message.tips.input.type')">
                                      <el-option v-for="item in machineKinds" :key="item" :label="item" :value="item"></el-option>
                                  </el-select>
                              </el-form-item>
                          
                              <el-form-item label="BMCIP">
                                  <el-input clearable v-model="queryCondition.bmcIP" :placeholder="$t('message.tips.input.BMCIP')"></el-input>
                              </el-form-item>
                              <el-form-item :label="$t('message.machineIP')">
                                  <el-input clearable v-model="queryCondition.businessIP" :placeholder="$t('message.tips.input.machineIP')"></el-input>
                              </el-form-item>
                          </el-col>
                          <el-col :span="12" >
                              <el-form-item :label="$t('message.owner')">
                                  <el-input clearable v-model="queryCondition.owner" :placeholder="$t('message.tips.input.owner')"></el-input>
                              </el-form-item>
                              <el-form-item :label="$t('message.machineUser')">
                                  <el-input clearable v-model="queryCondition.user" :placeholder="$t('message.tips.input.machineUser')"></el-input>
                              </el-form-item>
                          
                              <el-form-item :label="$t('message.address')">
                                  <el-input clearable v-model="queryCondition.virtualMachineIP" :placeholder="$t('message.tips.input.address')"></el-input>
                              </el-form-item>
                          </el-col>
                      </el-row>
                      <el-form-item align="right">
                          <el-button type="primary" @click="query()">{{$t('message.query')}}</el-button>
                          <el-button @click="queryCondition={}">{{$t('message.reset')}}</el-button>
                      </el-form-item>
                  </el-form>
              </el-collapse-item>
          </el-collapse>
          `
  };
  ```

* 注册组件

  组件注册有两种，一种是全局注册，一种是局部注册。由于全局注册的组件影响范围较广，所以一般采用局部注册。本项目中的条件查询模块就是采用局部注册，其代码如下

  ```vue
  new Vue({
      el: '#app',
      components: {
      	"query-block": queryBlock
      }
  })
  ```

* 使用组件

  使用组件很简单，只需要像正常的 html 元素调用即可。如下是对条件查询组件的调用。

  ```vue
  <query-block :machine-kinds="machineKinds" v-on:get-data="getData"></query-block>
  ```

* 组件通信

  * 通过 prop 向子组件传值

    在子组件中定义 prop 元素，这样父组件就可以通过定义的 prop 元素将父组件的值传递到子组件中，子组件使用这个值就像使用 data 元素里面的变量一样。需要注意的是，prop 元素在父组件的 html 相关代码中需要转成小写字母加横线的写法。如下代码是条件查询组件的 prop 传值

    ```vue
    <query-block :machine-kinds="machineKinds"></query-block>
    ```

    ```vue
    var queryBlock = {
        props: ["machineKinds"]   // 也可以写成 props: ["machine-kinds"]
    }
    ```

  * 通过 $emit 函数向父组件传值

    在子组件中，调用 emit 函数，传入一个标志符和相应的值，然后在父组件中通过传递的标识符获取相应的值。如下所示，子组件传递 `get-data` 标识符，其值为子组件的 `queryCondition`，在父组件中通过`v-on:get-data` 调用相应的函数获取子组件传递的值。

    ```vue
    this.$emit("get-data", this.queryCondition);
    ```

    ```vue
    <query-block v-on:get-data="getData"></query-block>
    
    <script>
    new Vue({
        el: '#app',
        method: {
            getData(queryCondition) {
                this.queryCondition = this.copyObject(queryCondition);
                this.currentPage = 1;
                this.getMachines(this.queryCondition);
            }
        }
    })
    </script>
    ```

关于组件更多的内容，请参考 [vue 官网](https://cn.vuejs.org/v2/guide/components.html)

## 用户 token 认证

这个主要是用在登录模块中的，主要用于判断用户是否登录。同时，由于系统只有一个用户，所以，也用这个 token 进行了权限的控制（对机器的增、删、改的权限控制）。

* token 的产生

  在用户登录成功是，在后台产生一个 token 返回给前台，后续前台发起相关请求时携带此 token 进行身份验证。出于简便的目的，直接用 UUID 的方式产生随机 token

  ```java
  String token = UUID.randomUUID().toString();
  ```

* token 的维护

  * 用户退出登录时，清除 token

    ```java
    @RequestMapping(value = "/logout", method = RequestMethod.POST)
    public void logout(HttpServletRequest request, HttpServletRequest reponse)
    {
        String token = request.getHeader("token");
        TokenUtil.removeToken(token);
    }
    ```

  * 用户在登录后每发起一个请求，就更新 token 的时间，否则可能会出现用户在操作（非退出登录）过程中突然 token 失效导致无法使用系统

    ```java
    public static void updateTime(String token)
    {
        tokenMap.replace(token, new Date());
    }
    ```

  * 前台在获取到后台返回的 token 时，将 token 保存到 cookie 中

    ```javascript
    localStorage.setItem("token", response.data);
    ```

* 前台发起请求携带 token

  本项目使用的 axios 向后台发送请求，axios 允许在所有请求的头部中增加相关信息。如下代码是在请求头中加入 token 信息，这样设置之后，所有的 axios 请求都会在 `Request Headers`  加入 token 信息。

  ```javascript
  // 设置请求头中增加token，用于认证是否登录
  axios.interceptors.request.use(function (config) {
      config.headers.token = localStorage.getItem("token");
      return config;
  }, function (error) {
      return Promise.reject(error);
  });
  ```

  关于 axios 的详细使用可以参考[在线文档](http://www.axios-js.com/zh-cn/docs/)

* 后台对 token 的校验

  由于对登录过后的用户发起的每个请求都需要更新 token 时间，所有最简单的做法是增加一个过滤器，拦截所有的请求，对每个请求中的 token 进行判断，存在系统中则进行时间的更新。同时，也可以在过滤器中对 token 的有效性进行校验。判断 token 的有效性主要是该 token 是否存在系统中以及是否过期。如果 token 不合法，则抛异常，这样就不会执行真正的业务代码。

  ```java
  private void validateToken(String token)
  {
      if (TokenUtil.isTokenExpired(token))
      {
          throw new RuntimeException("The token is invalid");
      }
  }
  ```

  ```java
  public static boolean isTokenExpired(String token)
  {
      if (StringUtils.isEmpty(token))
      {
          throw new RuntimeException("The token is invalid");
      }
  
      Date date = tokenMap.get(token);
      if (date == null)
      {
          throw new RuntimeException("The token is invalid");
      }
  
      Date currentDate = new Date();
      if (date.getTime() + DURATION_TIME < currentDate.getTime())
      {
          return true;
      }
  
      return false;
  }
  ```

  由于并不是所有的请求都要求登录，所以也并不是所有的请求都需要进行 token 的校验，因此，在进行 token   校验前，需要对请求进行判断，只有对机器的增删改的请求才进行校验，查询操作没有权限控制，不用进行 token 校验。

  ```java
  private boolean needAuth(HttpServletRequest request)
  {
      String requestURI = request.getRequestURI();
      String requestMethod = request.getMethod();
      if (StringUtils.equals("/machine", requestURI))
      {
          if (StringUtils.equals("POST", requestMethod)
              || StringUtils.equals("PUT", requestMethod)
              || StringUtils.equals("DELETE", requestMethod))
          {
              return true;
          }
  
          return false;
      }
  
      if (StringUtils.equals("/virtualMachine", requestURI))
      {
          return true;
      }
  
      return false;
  }
  ```

## 国际化

国际化主要涉及两方面，一个是使用 Element 框架组件自身的国际化，另一个是业务产生所需的国际化。

* 业务代码国际化

  国际化主要使用的是 vue 的插件 `vue-i18n`, 需要引入`<script src="https://unpkg.com/vue-i18n/dist/vue-i18n.js"></script>` 文件。

  其定义如下

  ```javascript
  const messages = {
      zh: {
          message: {
              delete: "删除",
          }
      },
      en: {
          message: {
              delete: "delete",
          }
      }
  };
  
  const i18n = new VueI18n({
      locale: "zh", // set locale
      messages, // set locale messages
  });
  
  new Vue({
      el: '#app',
      i18n,
      data: {
          
      }
  })
  ```

  使用时，分三种情况，有不同的用法

  * html 元素

    格式：

    ```vue
    {{$t('message.item')}}
    ```

    例子：

    ```vue
    <el-button type="primary" @click="login()">{{$t('message.login')}}</el-button>
    ```

  * Element元素传值

    格式：

    ```vue
    $t('message.item')
    ```

    例子：

    ```vue
    <el-input v-model="user.name" :placeholder="$t('message.tips.input.username')" ></el-input>
    ```

  * 函数

    格式：

    ```vue
    this.$t('message.item')
    ```

    例子：

    ```javascript
    this.$alert(errorMessage, this.$t('message.error'), {
        confirmButtonText: this.$t('message.confirm'),
        type: "error"
    });
    ```

  关于 vue-i18n 的更多内容，可以参考 [vue-i18n官网](http://kazupon.github.io/vue-i18n/zh/introduction.html)

* Element 组件国际化

  Element 组件默认使用中文，当将语言切换成英文时，需要将 Element 组件的语言也切换成英文。需要引入两个 Element 国际化相关的文件

  ```javascript
  <script src="https://unpkg.com/element-ui/lib/umd/locale/zh-CN.js"></script>
  <script src="https://unpkg.com/element-ui/lib/umd/locale/en.js"></script>
  ```

  在定义业务代码国际化时，将 Element 组件的国际化包含进去。如下所示

  ```javascript
  const messages = {
      zh: {
          message: {
              delete: "删除",
          },
          ...ELEMENT.lang.zhCN
      },
      en: {
          message: {
              delete: "delete",
          },
          ...ELEMENT.lang.en
      }
  };
  
  const i18n = new VueI18n({
      locale: "zh", // set locale
      messages, // set locale messages
  });
  
  // very important for element i18n
  Vue.use(ELEMENT, {
      i18n: (key, value) => i18n.t(key, value)
  });
  ```

  关于 Element 框架国际化的内容，请浏览 [Element 官网](https://element.eleme.cn/#/zh-CN/component/i18n)

* 语言间切换

  vue-i18n 提供了 `$i18n.locale` 变量来设置国际化语言。因此可以通过改变这个变量的值来达到切换语言的目的。代码如下

  ```vue
  <el-button type="text" @click="changeLanguaue('zh')" :disabled="$i18n.locale=='zh'">中文</el-button>
  <el-button type="text" @click="changeLanguaue('en')" :disabled="$i18n.locale=='en'">English</el-button>
  ```

  ```javascript
  changeLanguaue(lang) {
      this.$i18n.locale = lang;
  }
  ```

## 其他问题

请见：[一个微型项目中期总结](https://guozhchun.github.io/vue-springboot-demo-sumup-1/) 中 「遇到的问题及解决方案」章节

# 参考资料

1. vue 官网：[https://cn.vuejs.org/v2/guide/](https://cn.vuejs.org/v2/guide/)
2. axios 在线文档：[http://www.axios-js.com/zh-cn/docs/](http://www.axios-js.com/zh-cn/docs/)
3.  vue-i18n 官网：[http://kazupon.github.io/vue-i18n/zh/introduction.html](http://kazupon.github.io/vue-i18n/zh/introduction.html)
4. Element 官网：[https://element.eleme.cn/#/zh-CN/component/i18n](https://element.eleme.cn/#/zh-CN/component/i18n)

# 附录

项目地址：[https://github.com/guozhchun/vue-springboot-demo](https://github.com/guozhchun/vue-springboot-demo)