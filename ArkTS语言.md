# ArkTS语言

ArkTS是HarmonyOS优选的主力应用开发语言。

ArkTS围绕应用开发在TypeScript（简称TS）生态基础上做了进一步扩展，继承了TS的所有特性，是TS的超集。

当前，ArkTS在TS的基础上主要扩展了如下能力：

- **基本语法**：ArkTS定义了声明式UI描述、自定义组件和动态扩展UI元素的能力，再配合ArkUI开发框架中的系统组件及其相关的事件方法、属性方法等共同构成了UI开发的主体。

- **状态管理**：ArkTS提供了多维度的状态管理机制。
  - 在UI开发框架中，与UI相关联的数据可以在组件内使用，也可以在不同组件层级间传递，比如父子组件之间、爷孙组件之间，还可以在应用全局范围内传递或跨设备传递。
  - 从数据的传递形式来看，可分为只读的单向传递和可变更的双向传递。开发者可以灵活的利用这些能力来实现数据和UI的联动。

- **渲染控制**：ArkTS提供了渲染控制的能力。
  - `条件渲染`可根据应用的不同状态，渲染对应状态下的UI内容
  - `循环渲染`可从数据源中迭代获取数据，并在每次迭代过程中创建相应的组件。
  - `数据懒加载`从数据源中按需迭代数据，并在每次迭代过程中创建相应的组件。

## 基本语法

![ArkTS的基本组成](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20231013151059.31410059096453875311411412779760:50001231000000:2800:5E294F9487B72D4F22BD159CFDC5F335867C7B7998D5A501541F9D56C9BF54A7.png?needInitFileName=true?needInitFileName=true)

- 装饰器： 用于装饰类、结构、方法以及变量，并赋予其特殊的含义。如上述示例中@Entry、@Component和@State都是装饰器，@Component表示自定义组件，@Entry表示该自定义组件为入口组件，@State表示组件中的状态变量，状态变量变化会触发UI刷新。
- UI描述：以声明式的方式来描述UI的结构，例如build()方法中的代码块。
- 自定义组件：可复用的UI单元，可组合其他组件，如上述被@Component装饰的struct Hello。
- 系统组件：ArkUI框架中默认内置的基础和容器组件，可直接被开发者调用，比如示例中的Column、Text、Divider、Button。
- 属性方法：组件可以通过链式调用配置多项属性，如fontSize()、width()、height()、backgroundColor()等。
- 事件方法：组件可以通过链式调用设置多个事件的响应逻辑，如跟随在Button后面的onClick()。

## 组件

在ArkUI中，UI显示的内容均为组件，由框架直接提供的称为系统组件，由开发者定义的称为自定义组件。

自定义组件具有以下特点：

- 可组合：允许开发者组合使用系统组件、及其属性和方法。
- 可重用：自定义组件可以被其他组件重用，并作为不同的实例在不同的父组件或容器中使用。
- 数据驱动UI更新：通过状态变量的改变，来驱动UI的刷新。

### 自定义组件的基本结构

```ts
@Entry
@Component
struct MyComponent {
  build() {
    //
  }
}
```

1. struct：自定义组件基于struct实现，struct + 自定义组件名 + {...}的组合构成自定义组件，不能有继承关系。
2. @Component：@Component装饰器仅能装饰struct关键字声明的数据结构。

    - struct被@Component装饰后具备组件化的能力，需要实现build方法描述UI，一个struct只能被一个@Component装饰。

3. build()函数：build()函数用于定义自定义组件的声明式UI描述，自定义组件必须定义build()函数。

4. @Entry：@Entry装饰的自定义组件将作为UI页面的入口。

    - 在单个UI页面中，最多可以使用@Entry装饰一个自定义组件

### 成员函数/变量

自定义组件除了必须要实现build()函数外，还可以实现其他成员函数，**成员函数**具有以下约束：

- 不支持静态函数。
- 成员函数的访问始终是私有的。

自定义组件可以包含**成员变量**，成员变量具有以下约束：

- 不支持静态成员变量。
- 所有成员变量都是私有的，变量的访问规则与成员函数的访问规则相同。
- 自定义组件的成员变量本地初始化有些是可选的，有些是必选的（与是否从父组件传递参数有关系）。

### build()函数

1. @Entry装饰的自定义组件，其build()函数下的根节点唯一且必要，**且必须为容器组件**，其中ForEach禁止作为根节点
2. @Component装饰的自定义组件，其build()函数下的根节点唯一且必要，**可以为非容器组件**，其中ForEach禁止作为根节点。
3. 不允许声明本地变量

    ```ts
    build() {
      // 反例：不允许声明本地变量
      let a: number = 1;
    }    
    ```

4. 不允许在UI描述里直接使用console.info，但允许在方法或者函数里使用
5. 不允许创建本地的作用域

    ```ts
    build() {
      // 反例：不允许本地作用域
      {
        ...
      }
    }
    ```

6. 不允许调用除了被@Builder装饰以外的方法，允许系统组件的参数是TS方法的返回值

    ```ts
    @Component
    struct ParentComponent {
      doSomeCalculations() {
      }

      calcTextValue(): string {
        return 'Hello World';
      }

      @Builder doSomeRender() {
        Text(`Hello World`)
      }

      build() {
        Column() {
          // ❌反例：不能调用没有用@Builder装饰的方法
          this.doSomeCalculations();
          // ✅正例：可以调用
          this.doSomeRender();
          // ✅正例：参数可以为调用TS方法的返回值
          Text(this.calcTextValue())
        }
      }
    }
    ```

7. 不允许switch语法，如果需要使用条件判断，请使用if。
8. 不允许使用表达式（参数里可以使用）

    ```ts
    build() {
      Column() {
        // ❌反例：不允许使用表达式
        (this.aVar > 10) ? Text('...') : Image('...')
        // ✅正例：可以
        Text(this.aVar > 10 ? 'A' : 'B')
      }
    }
    ```

### 页面和自定义组件生命周期

自定义组件和页面的关系：

- 自定义组件：@Component装饰的UI单元，可以组合多个系统组件实现UI的复用。
- 页面：即应用的UI页面。
  - 可以由一个或者多个自定义组件组成
  - @Entry装饰的自定义组件为页面的入口组件，即页面的根节点，一个页面有且仅能有一个@Entry。
  - **只有被@Entry装饰的组件才可以调用页面的生命周期**。

页面生命周期，即被@Entry装饰的组件生命周期，提供以下生命周期接口：

- onPageShow：页面每次显示时触发。
- onPageHide：页面每次隐藏时触发一次。
- onBackPress：当用户点击返回按钮时触发。

组件生命周期，即一般用@Component装饰的自定义组件的生命周期，提供以下生命周期接口：

- aboutToAppear：组件即将出现时回调该接口，具体时机为在创建自定义组件的新实例后，在执行其build()函数之前执行。
- aboutToDisappear：在自定义组件即将析构销毁时执行。

生命周期流程如下图所示，下图展示的是被@Entry装饰的组件（首页）生命周期

![生命周期](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20231016160223.99199945095842886884636368090803:50001231000000:2800:019E4D38024DDC3B25A19EE8BEF753B3FB089891518C310555FB21C5117F6BF7.png?needInitFileName=true?needInitFileName=true)

## @Builder装饰器：自定义构建函数
