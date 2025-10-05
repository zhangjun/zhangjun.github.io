---
title: design pattern
excerpt: UML设计模式基础概念详解，包括依赖、泛化、实现、关联、聚合、组合等关系的定义、表示方法和C++代码示例，帮助理解面向对象设计原则。
---

## UML
- 依赖（Dependency）：元素A的变化会影响元素B, B依赖A. 用带箭头的虚线表示Dependency关系，箭头指向被依赖元素。
    ```cpp
    class Person {
        void buy(Car car) {
        ...
        }
    }
    ```
    ![image](https://user-images.githubusercontent.com/1312389/218317998-ef2f20c2-7476-466d-a45a-2330763e37af.png)

- 泛化（Generalization）：通常所说的继承（特殊个体 is kind of 一般个体）关系. 带空心箭头的实线线表示Generalization关系，箭头指向一般个体。

- 实现（Realize）: 元素A定义一个约定，元素B实现这个约定
- 关联（Association）：元素间的结构化关系，是一种弱关系，被关联的元素间通常可以被独立的考虑。uml中用实线表示Association关系，箭头指向被依赖元素。

    ![image](https://user-images.githubusercontent.com/1312389/218318328-ccd536a0-54a3-4bb0-a4e8-504db37fa235.png)

- 聚合（Aggregation）：关联关系的一种特例，表示部分和整体（整体 has a 部分）的关系. 带空心菱形头的实线表示Aggregation关系，菱形头指向整体。

    ![image](https://user-images.githubusercontent.com/1312389/218318177-87c15ff7-a741-4ab4-857e-351b4bcb7ac5.png)

- 组合（Composition）：组合是聚合关系的变种，表示元素间更强的组合关系。如果是组合关系，如果整体被破坏则个体一定会被破坏，而聚合的个体则可能是被多个整体所共享的，不一定会随着某个整体的破坏而被破坏。uml中用带实心菱形头的实线表示Composition关系，菱形头指向整体。

    ![image](https://user-images.githubusercontent.com/1312389/218318213-a96cdb17-a44c-435e-a188-7216ab455ebf.png)
