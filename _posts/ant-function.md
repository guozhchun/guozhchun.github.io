---
title: ant 类函数调用功能
date: 2018-09-23 15:26:26
tags: ant
categories: ant
---

# 前言

ant 是一种打包脚本语言，能极大地方便程序的编译打包工作。但是在平常的开发过程中，经常会遇到一些打包流程，除了一小部分不一样，其他都是一样的。这种情况在编程语言中一般会通过抽取调用函数来减少重复代码，但是在 ant 中并没有函数这种功能。那应该如何来减少重复的代码呢？本文将通过宏定义 `macrodef` 和 `antcall` 两种方式来实现 ant 中类似函数的功能。

<!-- more -->

# 宏定义 macrodef

宏定义的方式主要是通过在宏内部定义变量，并将主体的实现放在 `sequential` 块中来实现的。其格式如下

```xml
<!-- 定义宏 -->
<macrodef name="macroName">
    <!-- 定义宏属性，值由调用宏的地方传入 -->
    <attribute name="attributeName" />
    ...
	
    <!-- 宏主体实现内容，此处可通过 @{attributeName} 获取传入的 attributeName 变量的值 -->
    <sequential>
        ...
    </sequential>
</macrodef>

<!-- 调用宏, macroName 是定义的宏的 name 属性的值，attributeName 是宏内部定义的属性 -->
<macroName attributeName="..." />
<macroName attributeName="..." />
```

一个简单的例子如下，假设有多个打包任务，除了打包的路径和打包的名字不同，其他步骤是一样的，此时使用宏定义实现的代码如下

```xml
<project name="testFunction" default="build" basedir=".">
    <macrodef name="package">
        <attribute name="srcDir" default="defaultDir" />
        <attribute name="packageName" default="defaultName" />

        <sequential>
            <echo message="====================================" />
            <echo message="prepare making a package in source code directory @{srcDir}" />
            <echo message="making a package with name @{packageName}" />
            <echo message="clean the environment after making a package" />
        </sequential>
    </macrodef>

    <target name="build">
        <package />
        <package srcDir="temp" packageName="tempName" />
        <package srcDir="release" packageName="releaseName" />
    </target>
</project>
```

程序输出如下

![宏定义输出](/images/ant-function-macrodef-output.png)

# antcall

antcall 的方式主要是在调用 target 时可以主动传入 param 属性，在这个属性标签中可以为相应的属性设置特定的值。其格式如下

```xml
<!-- 需要被重复调用的 target 定义，内部实现可以用 ${paramName} 来获取传入的 paramName 的值 -->
<target name="targetName">
    ...
</target>

<!-- 重复调用其他的 target -->
<target name="build">
    <antcall target="targetName">
        <param name="param1" value="value1"/>
        <param name="param2" value="value2"/>
        ...
    </antcall>

    <antcall target="targetName">
        <param name="param1" value="value1111"/>
        <param name="param2" value="value2222"/>
        ...
    </antcall>
    ...
</target>
```

一个简单的例子如下，为了便于比较，此例子实现的功能跟宏定义 macrodef 中的例子一样：有多个打包任务，除了打包的路径和打包的名字不同，其他步骤是一样的。代码如下

```xml
<project name="testFunction" default="build" basedir=".">
    <target name="package">
        <!-- 为 srcDir 和 packageName 设置默认值 -->
        <property name="srcDir" value="defaultDir" />
        <property name="packageName" value="defaultName" />
        
        <echo message="====================================" />
        <echo message="prepare making a package in source code directory ${srcDir}" />
        <echo message="making a package with name ${packageName}" />
        <echo message="clean the environment after making a package" />
    </target>

    <target name="build">
        <antcall target="package" />
        
        <antcall target="package">
            <param name="srcDir" value="temp"/>
            <param name="packageName" value="tempName"/>
        </antcall>

        <antcall target="package">
            <param name="srcDir" value="release"/>
            <param name="packageName" value="releaseName"/>
        </antcall>
    </target>
</project>
```

程序输出如下

![antcall 输出](/images/ant-function-antcall-output.png)

# 比较

宏定义和 antcall 两种方式都能减少重复的代码，实现类似函数调用的功能。两者各有优缺点：宏定义需要在内部额外定义需要传入的属性名称，同时可以为这个属性却设置默认值，在调用宏定义时代码相对而言比较少，看起来也比较清晰；相对而言，antcall 的重复调用的 target 跟其他普通的 target 并没有区别，这相对比较简单，只是在调用的地方需要通过 `param` 标签来指定每个属性的名称和值，在参数多的时候，这很容易造成代码的不雅观，而且当需要为属性设置默认值时，需要额外通过 `property` 标签进行设值。