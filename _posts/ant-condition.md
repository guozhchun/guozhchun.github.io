---
title: ant条件判断
date: 2018-07-08 11:13:44
tags: ant
categories: ant
---

# 前言

ant是一种常用的打包构建工具，支持在构建过程中根据属性值判断选择构建的的步骤，本文将以condition中的equals标签为例，说明ant中的条件判断的用法。

<!-- more -->

# 简单属性条件判断

简单属性的条件判断是在target标签中增加 if 元素判断，且判断的依据是 if 元素中的属性是否设置了值，如果设置了值，不管值是什么内容（即使是false），都会执行该target，如果没有设置该值，则不执行该target。需要注意的是，在target中的条件判断只能是一个，如果需要多个条件判断，需要将其写在condition标签中，然后将condition标签绑定在一个属性中，再用这个属性写在target中进行判断。

## 格式

```xml
<target name="targetName" if="propertionCondition">
	...
</target>
```

其中propertionCondition是一个属性的名称

## 例子

以下例子中，有三个判断：isTrue属性，isWindows属性，isRed属性。

* isTrue属性：由于isTrue属性设置了值false，所以在`<target name="propertyTrue" if="isTrue">`的判断中，其为真，会执行该target
* isWindows属性：由于os属性的值为windows，故`<equals arg1="windows" arg2="${os}" />`条件成立，给isWindows属性设置了值，默认是true（从windows的target执行输出结果可以证实），所以在`<target name="windows" if="isWindows">`的判断中，条件成立，会执行windows的target。而在`<target name="notWindows" unless="isWindows">`的判断中，条件不成立，不会执行notWindows的target
* isRed属性：由于color属性的值为green，故`<equals arg1="red" arg2="${color}" />`条件不成立，不会给isRed属性设置值（从notRed的target的执行输出结果可以证实），所以在`<target name="red" if="isRed">`的判断中，条件不成立，不会执行red的target。而在`<target name="notRed" unless="isRed">`的判断中，条件成立，执行notRed的target

```xml
<project name="testAntCondition" default="build" basedir=".">
    <property name="os" value="windows" />
    <property name="color" value="green" />
    <property name="isTrue" value="false" />

    <!-- 如果arg1和arg2相等，则设置isWindows的值（默认是true），否则不设置值 -->
    <condition property="isWindows" >
        <equals arg1="windows" arg2="${os}" />
    </condition>

    <!-- 如果arg1和arg2相等，则设置isRed的值（默认是true），否则不设置值 -->
    <condition property="isRed">
        <equals arg1="red" arg2="${color}" />
    </condition>

    <!-- 此种if写法判断的是属性是否有设置值，如果设置了值，不管是什么值都会执行，不设置值则不执行设置 -->
    <!-- 由于isTrue设置了值（虽然是false），此时条件成立，所以这个target会被执行 -->
    <target name="propertyTrue" if="isTrue">
        <echo message="This is a target which name is propertyTrue" />
        <echo message="The isTrue value is ${isTrue}" />
    </target>

    <!-- isWindows设置了值，是true，所以，windows会执行，notWindows不会执行 -->
    <target name="windows" if="isWindows">
        <echo message="This is a target which name is windows" />
        <echo message="The isWindows value is ${isWindows}" />
    </target>
    <target name="notWindows" unless="isWindows">
        <echo message="This is a target which name is notWindows" />
        <echo message="The isWindows value is ${isWindows}" />
    </target>

    <!-- isRed没有设置值，所以，notRed会执行，red不会执行 -->
    <target name="red" if="isRed">
        <echo message="This is a target which name is red" />
        <echo message="The isRed value is ${isRed}" />
    </target>
    <target name="notRed" unless="isRed">
        <echo message="This is a target which name is notRed" />
        <echo message="The isRed value is ${isRed}" />
    </target>

    <target name="build">
        <antcall target="propertyTrue" />
        <antcall target="windows" />
        <antcall target="notWindows" />
        <antcall target="red" />
        <antcall target="notRed" />
    </target>
</project>
```

输出如下

![ant简单属性条件判断输出结果](/images/antSimpleCondition.png)

# 扩展属性条件判断

扩展属性的条件判断是在target标签中增加 if 元素判断，且判断的依据是 if 元素中的属性值是否是true，如果是true，则执行该target，否则不执行该target（不一定设置属性值是false，只要不是true的其他值都可以）。需要注意的是，在target中的条件判断只能是一个，如果需要多个条件判断，需要将其写在condition标签中，然后将condition标签绑定在一个属性中，再用这个属性写在target中进行判断。

## 格式

```xml
<target name="targetName" if="${propertionCondition}">
	...
</target>
```

其中propertionCondition是一个属性的名称

## 例子

以下例子中，有三个判断：isTrue属性，isWindows属性，isRed属性。

- isTrue属性：由于isTrue属性设置了值false，所以在`<target name="propertyTrue" if="isTrue">`的判断中，条件不成立，不会执行该target
- isWindows属性：由于os属性的值为windows，故`<equals arg1="windows" arg2="${os}" />`条件成立，给isWindows属性设置值为true（从windows的target执行输出结果可以证实），所以在`<target name="windows" if="isWindows">`的判断中，条件成立，会执行windows的target。而在`<target name="notWindows" unless="isWindows">`的判断中，条件不成立，不会执行notWindows的target
- isRed属性：由于color属性的值为green，故`<equals arg1="red" arg2="${color}" />`条件不成立，给isRed属性设置值为false（从notRed的target的执行输出结果可以证实），所以在`<target name="red" if="isRed">`的判断中，条件不成立，不会执行red的target。而在`<target name="notRed" unless="isRed">`的判断中，条件成立，执行notRed的target

```xml
<project name="testAntCondition" default="build" basedir=".">
    <property name="os" value="windows" />
    <property name="color" value="green" />
    <property name="isTrue" value="false" />

    <!-- 如果arg1和arg2相等，则设置isWindows的值为true，否则设置为false -->
    <condition property="isWindows" value="true" else="false">
        <equals arg1="windows" arg2="${os}" />
    </condition>

    <!-- 如果arg1和arg2相等，则设置isRed的值为true，否则设置为false -->
    <condition property="isRed" value="true" else="false">
        <equals arg1="red" arg2="${color}" />
    </condition>

    <!-- 此种if写法判断的是属性值是否是true，如果属性值是true则执行，否则不执行 -->
    <!-- 由于isTrue是false，此时条件不成立，所以这个target不会被执行 -->
    <target name="propertyTrue" if="${isTrue}">
        <echo message="This is a target which name is propertyTrue" />
        <echo message="The isTrue value is ${isTrue}" />
    </target>

    <!-- isWindows属性值是true，所以，windows会执行，notWindows不会执行 -->
    <target name="windows" if="${isWindows}">
        <echo message="This is a target which name is windows" />
        <echo message="The isWindows value is ${isWindows}" />
    </target>
    <target name="notWindows" unless="${isWindows}">
        <echo message="This is a target which name is notWindows" />
        <echo message="The isWindows value is ${isWindows}" />
    </target>

    <!-- isRed属性值是false，所以，notRed会执行，red不会执行 -->
    <target name="red" if="${isRed}">
        <echo message="This is a target which name is red" />
        <echo message="The isRed value is ${isRed}" />
    </target>
    <target name="notRed" unless="${isRed}">
        <echo message="This is a target which name is notRed" />
        <echo message="The isRed value is ${isRed}" />
    </target>

    <target name="build">
        <antcall target="propertyTrue" />
        <antcall target="windows" />
        <antcall target="notWindows" />
        <antcall target="red" />
        <antcall target="notRed" />
    </target>
</project>
```

输出如下

![ant扩展属性条件判断输出结果](/images/antExpansionsCondition.png)

# 比较

简单属性条件判断和扩展属性条件判都能让target根据一定条件选择是否执行该target，两者的目的和效果都一样，限制条件也一样：一个target只能有一个条件判断。唯一不同的是简单属性条件判断中的 if 元素写的是属性，而且是判断属性是否设置了值；而扩展属性条件判断中的 if 元素写的是属性值，而且是判断属性值是否是true。相比而言，简单属性条件判断比较简单，但是扩展属性条件判断更加灵活也更加容易理解。所以建议是使用中选择扩展属性条件判断方式。

# 扩展阅读

ant中支持的条件判断：http://ant.apache.org/manual/Tasks/conditions.html

# 参考资料

ant使用指导：http://ant.apache.org/manual/index.html