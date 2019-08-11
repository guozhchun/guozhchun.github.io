---
title: 一个微型项目中期总结
date: 2019-06-02 17:11:12
tags: [springboot, vue, mybatis]
categories: 其他
---

# 前言

之所以叫「微型项目」，是因为这个项目页面和功能都很单一，只满足基本功能的增删改查，并没有其他额外的考虑，如：国际化、安全性。因为做这个项目的初衷就是为了将部门内机器的使用情况进行汇总公开，方便后续的管理。为了能够快速完成开发，就只完成了基本的主要的功能，放弃了一些非功能性要求和交互性。这也是将其称为「微型项目」而不是「小型项目」的原因。

<!-- more -->

虽然，目前项目还没有全部开发完成，只完成了一部分主要功能，但是在这过程中也遇到了一些问题，担心积累到后面会遗忘，所以先在此做一部分回顾与总结。

# 需求

虽然是自己折腾，但是也得有个目的，以下功能就是本次项目需要实现的。

* 能够对物理机进行增删改查
* 能够对物理机上的虚拟机进行增删改查
* 能够通过某些查询条件（机器使用者，资产编号，机器挂账人等）进行查询得到结果
* 只有特定的人才能对机器进行增删改，普通人只能进行查询

# 选型

* CSS框架：~~Bootstrap~~ Element

  因为以前只对 Bootstrap 有过了解，觉得其封装了一些组件挺好的，不用自己调样式，所以，刚开始就想用 Bootstrap 来配合使用搭建页面。后面发现 Bootstrap 封装还是过于简单，一些弹窗什么的都需要自己另外写，太麻烦了。同时又了解到 Element 配合 Vue 封装了大量常用的组件，极大地方便了页面的开发，所以就从 Bootstrap 切换到 Element

* 前端框架：Vue

  其实现在项目中用的是公司自己封装过 Angularjs 的一个框架，理论上我用 Angularjs 能够更加快速地上手和开发，但是我却选择用 Vue，一个原因是想趁着这次折腾自己的机会学习下 Vue，另一个原因是项目组中曾经有想过要切换框架到 Vue，所以也可以提前了解准备下。

* 后端框架：Springboot

  选择这个的原因就很简单了，就是图方便快捷。直接配置 maven 就能运行，不能额外弄其他的东西。

* 数据库框架：MyBatis

  选择这个框架是因为可以直接写 sql，不用在 java 代码中动态拼接，也不用写很多的配置文件，看起来比较直观。

* 数据库：mysql

# 遇到的问题及解决方案

## Vue 整合到 Springboot 中

由于本次使用 Vue 进行开发是通过引用 CDN 文件的形式，不是通过模块化的方式进行操作的，而网上的 Vue 和 Springboot 的整合基本上都是基于模块化的 Vue 进行操作的。这就导致我不知道如何将两者进行整合运行。在进行了一番搜索后，知道 Spring 默认会读取 static 目录下的 index.html 文件，因此将 Vue 的文件命名为 index.html 然后放入 static 目录下，启动 springboot 即可通过浏览器访问。

## Vue 中函数调用函数问题

正常情况下，需要通过  `this`  来调用，如下所示

```javascript
submitVirtualMachine() {
    if (this.virtualMachineDialog.type == "add") {
        this.addVirtualMachine(this.virtualMachineDialog.virtualMachine);
    } else {
        this.updateVirtualMachine(this.virtualMachineDialog.virtualMachine);
    }
}
```

但是当发起异步请求的处理结果过程中，通过 `this` 也无法调用实例中的函数。此时，需要在发起异步请求前用一个变量将 `this` 对象保存起来，在对发起异步后得到的返回结果进行处理时，使用保存的 `this` 值的变量来调用函数。如下所示

```javascript
updateVirtualMachine(virtualMachine) {
    var _this = this;    // 保存 this 对象
    axios.put("/virtualMachine", virtualMachine).then(function(response) {
        if (response.data == "SUCCESS") {
            // 通过 _this 对象而不是 this 对象来调用
            // 通过 this 对象调用函数会报错
            _this.handleVirtualMachineSuccess();
        } else {
            _this.handleMachineError(response.data)
        }
    }).catch(function(error) {
        _this.handleMachineError("OPERATION_FAIL");
    });
}
```

## Vue 页面初始化问题

想要在页面加载时调用某个函数，可以将函数写在 mounted 中，如下所示

```javascript
mounted: function() {
    this.getAllMachines();
}
```

## axios.delete 请求报没有 requestbody 问题

使用`axios.post(url, jsonData)`、 `axios.put(url, jsonData)` 都没有问题，但是使用 `axios.delete(url, jsonData)` 就会报 `Required request body is missing`  的错误，需要使用 `axios.delete(url, {data: jsonData})` 来传递 request body 的内容。另外，如果需要使用类似 `http://IP:PORT/XXX?param1=x&param2=xx` 的请求传递 URL 参数，则需要使用 `axios.delete(url, {params: param})` 

## 本地后台调试问题

在 maven 配置文件中加入 `jvmArguments` 配置项，并将其值设为： `-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005`，然后在 IDE 上配置 IP 和端口号，即可启动调试。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <jvmArguments>
                    -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
                </jvmArguments>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 进展

当前已经以下两个内容的开发

* 能够对物理机进行增删改查
* 能够对物理机上的虚拟机进行增删改查

部分效果图如下

![首页](/images/machine-management-1.png)

![增加机器](/images/machine-management-2.png)

![增加虚拟机](/images/machine-management-3.png)

# 后续计划

后续计划继续完成以下内容开发

- 能够通过某些查询条件（机器使用者，资产编号，机器挂账人等）进行查询得到结果
- 只有特定的人才能对机器进行增删改，普通人只能进行查询

# 总结

* 表结构的设计很重要，在明确需求后，要先设计好数据结构。在开发前，需要先在脑海中或纸上预演一遍，看设计的表结构是否能够满足所有的流程。否则开发到一半时，需要修改表结构甚至推到重来，是很痛苦的事。
* 在进行技术选型时，除了自身原因的考虑（想要自己瞎折腾），要广泛搜索业界常用的搭配及使用方式，然后结合自身找到一个合适的框架进行开发，这样能够极大地提升开发效率。

# 附录

项目地址：[https://github.com/guozhchun/vue-springboot-demo](https://github.com/guozhchun/vue-springboot-demo)