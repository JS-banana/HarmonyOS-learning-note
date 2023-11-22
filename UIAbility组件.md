# UIAbility组件

UIAbility组件是一种包含UI界面的应用组件，主要用于和用户交互

- UIAbility组件是系统调度的基本单元，为应用提供绘制界面的窗口
- 一个UIAbility组件中可以通过多个页面来实现一个功能模块
- 每一个UIAbility组件实例，都对应于一个最近任务列表中的任务

> 需要在module.json5配置文件的abilities标签中声明UIAbility的名称、入口、标签等相关信息

## 生命周期

UIAbility的生命周期包括Create、Foreground、Background、Destroy四个状态

![UIAbility生命周期状态](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20231013150932.16345942681464876975140848517205:50001231000000:2800:FAB1F72B3F479F70634B0E15FD9666B6AB2CD8AF56E3FE023ED7A309A31AB93A.png?needInitFileName=true?needInitFileName=true)

1. Create状态

    UIAbility实例创建完成时触发，系统会调用onCreate()回调

    - 可以在该回调中进行应用初始化操作，例如变量定义资源加载等

2. Foreground和Background状态

    Foreground和Background状态分别在UIAbility实例切换至前台和切换至后台时触发，对应于onForeground()回调和onBackground()回调

3. Destroy状态

    Destroy状态在UIAbility实例销毁时触发

### WindowStageCreate和WindowStageDestroy状态

UIAbility实例创建完成之后，在进入Foreground之前，系统会创建一个WindowStage

WindowStage创建完成后会进入onWindowStageCreate()回调，可以在该回调中设置UI界面加载、设置WindowStage的事件订阅

![WindowStageCreate和WindowStageDestroy状态](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20231013150932.63354127475238565550168838734582:50001231000000:2800:35AC14D3A2A6FB85BE83F9A33EA2F0823AA79D0E35C27327182BC88B55EDC0BA.png?needInitFileName=true?needInitFileName=true)

## UIAbility组件启动模式

UIAbility的启动模式是指UIAbility实例在启动时的不同呈现状态。

系统提供了三种启动模式：

- singleton（单实例模式）——默认情况下的启动模式
- standard（标准实例模式）
- specified（指定实例模式）

需要在module.json5配置文件中的"launchType"字段配置

1. singleton启动模式

    每次调用startAbility()方法时，如果应用进程中该类型的UIAbility实例已经存在，则复用系统中的UIAbility实例。

    - 只存在唯一一个该UIAbility实例
    - 应用的UIAbility实例已创建，再次调用startAbility()方法启动该UIAbility实例，此时只会进入该UIAbility的`onNewWant()`回调

2. standard启动模式

    每次调用startAbility()方法时，都会在应用进程中创建一个新的该类型UIAbility实例

    - 可以看到有多个该类型的UIAbility实例

3. specified启动模式

    存在标记的实例时直接复用，不存在时则新建

    - 如：文档应用中每次新建文档都能新建一个文档实例，重复打开一个已保存的文档打开的都是同一个文档实例

## UIAbility组件基本用法

指定UIAbility的启动页面以及获取UIAbility的上下文UIAbilityContext。

### 指定UIAbility的启动页面

应用中的UIAbility在启动过程中，需要指定启动页面，否则应用启动后会因为没有默认加载页面而导致白屏。

```ts
import UIAbility from '@ohos.app.ability.UIAbility';
import Window from '@ohos.window';

export default class EntryAbility extends UIAbility {
    onWindowStageCreate(windowStage: Window.WindowStage) {
        // Main window is created, set main page for this ability
        windowStage.loadContent('pages/Index', (err, data) => {
            // ...
        });
    }

    // ...
}
```

### 获取UIAbility的上下文信息

UIAbility类拥有自身的上下文信息，该信息为UIAbilityContext类的实例

1. 在UIAbility中可以通过this.context获取UIAbility实例的上下文信息
2. 在页面中获取UIAbility实例的上下文信息

```ts
import UIAbility from '@ohos.app.ability.UIAbility';

export default class EntryAbility extends UIAbility {
    onCreate(want, launchParam) {
        // 获取UIAbility实例的上下文
        let context = this.context;

        // ...
    }
}
```

## UIAbility组件与UI的数据同步

两种方式：

1. EventHub：基于发布订阅模式来实现，事件需要先订阅后发布，订阅者收到消息后进行处理。
2. globalThis：ArkTS引擎实例内部的一个全局对象，在ArkTS引擎实例内部都能访问。

### EventHub

eventHub 可以通过上下文获取

逻辑和用法与 vue 的 $event 非常类似

```ts
// 执行订阅操作
this.context.eventhub.on('event1', this.func1);

// 通过eventHub.emit()方法触发该事件
this.context.eventHub.emit('event1', 1);

// context为UIAbility实例的AbilityContext
this.context.eventHub.off('event1');
```

### globalThis

globalThis是ArkTS引擎实例内部的一个全局对象，引擎内部的UIAbility/ExtensionAbility/Page都可以使用，因此可以使用globalThis全局对象进行数据同步。

![使用globalThis进行数据同步](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20231013150933.76047542222691001213152941421944:50001231000000:2800:59000B39A7BD4C539DD6A3C0B9FBD2444595946E1C70270277895DD7FDBA9D74.png?needInitFileName=true?needInitFileName=true)

> 在UIAbility组件和自定义UI组件中都可以直接访问 globalThis 对象

```ts
// 1. UIAbility
import UIAbility from '@ohos.app.ability.UIAbility'

export default class EntryAbility extends UIAbility {
    onCreate(want, launch) {
        globalThis.entryAbilityWant = want;
    }
}

// 2. UI组件
@Entry
@Component
struct Index {
  aboutToAppear() {
    let entryAbilityWant = globalThis.entryAbilityWant;
  }
}
```

## UIAbility组件间交互（设备内）

UIAbility是系统调度的最小单元。在设备内的功能模块之间跳转时，会涉及到启动特定的UIAbility，该UIAbility可以是应用内的其他UIAbility，也可以是其他应用的UIAbility（例如启动三方支付UIAbility）。

#### 1. 启动应用内的UIAbility

调用方A，被调用方B

在A中，通过 startAbility 方法实现调用

```ts
// context为调用方UIAbility的AbilityContext
let wantInfo = {
    bundleName: 'com.example.b',
    abilityName: 'com.example.b.MainAbility'
     parameters: { // 自定义信息
        info: '来自EntryAbility Index页面',
    },
    // ...
}

this.context.startAbility(wantInfo).then(() => {
    // ...
}).catch((err) => {
    // ...
})

```

在B中，可以在 onCreate 方法中接受

```ts
import UIAbility from '@ohos.app.ability.UIAbility';
import Window from '@ohos.window';

export default class FuncAbility extends UIAbility {
    onCreate(want, launchParam) {
    // 接收调用方UIAbility传过来的参数
        let funcAbilityWant = want;
        let info = funcAbilityWant?.parameters?.info;
        // ...
    }
}
```

在B中，业务完成后的销毁逻辑，使用 terminateSelf() 方法实现

```ts
// context为需要停止的UIAbility实例的AbilityContext
this.context.terminateSelf((err) => {
    // ...
});
```

#### 2. 启动应用内的UIAbility并获取返回结果

与上面类似，方法不同

在A中，使用startAbilityForResult()接口启动

```ts
this.context.startAbilityForResult(wantInfo).then((data) => {
    // ...
}).catch((err) => {
    // ...
})
```

在B中，进行参数传递，在需要停止自身时，调用terminateSelfWithResult()方法，入参为需要返回给A的信息。

```ts
let abilityResult = {
    resultCode: RESULT_CODE,
    want: {
        bundleName: 'com.example.myapplication',
        abilityName: 'FuncAbility',
        moduleName: 'module1',
        parameters: {
            info: '来自FuncAbility Index页面',
        },
    },
}
this.context.terminateSelfWithResult(abilityResult, (err) => {
    // ...
});
```

在B停止自身后，A通过startAbilityForResult()方法回调接收被B返回的信息

```ts
this.context.startAbilityForResult(want).then((data) => {
    if (data?.resultCode === RESULT_CODE) {
        // 解析被调用方UIAbility返回的信息
        let info = data.want?.parameters?.info;
        // ...
    }
}).catch((err) => {
    // ...
})
```

#### 3. 启动其他应用的UIAbility

启动其他应用的UIAbility，通常用户只需要完成一个通用的操作（例如：需要选择一个文档应用来查看某个文档的内容信息）

推荐使用隐式Want启动。系统会根据调用方的want参数来识别和启动匹配到的应用UIAbility

- 显式Want启动：启动一个确定应用的UIAbility
- 隐式Want启动：根据匹配条件由用户选择启动哪一个UIAbility
  -在调用startAbility()方法时，其入参want中指定一些参数信息，然后由系统去分析want，并帮助找到合适的UIAbility来启动。
  - 当需要拉起其他应用的UIAbility时，开发者通常不知道用户设备中应用的安装情况，也无法确定目标应用的bundleName和abilityName，通常使用隐式Want启动方式。

显式Want启动流程：

1. 将多个待匹配的文档应用安装到设备，在其对应UIAbility的module.json5配置文件中，配置skills的entities字段和actions字段
2. 在调用方want参数中的entities和action需要被包含在待匹配UIAbility的skills配置的entities和actions中

系统匹配到符合entities和actions参数条件的UIAbility后，会弹出选择框展示匹配到的UIAbility实例列表供用户选择使用

#### 4. 启动其他应用的UIAbility并获取返回结果

同上，使用 startAbilityForResult、terminateSelfWithResult

#### 5. 启动UIAbility的指定页面

一个UIAbility可以对应多个页面，在不同的场景下启动该UIAbility时需要展示不同的页面

在A中，指定跳转B应用的指定页面参数

```ts
let wantInfo = {
    // ...
    parameters: { // 自定义参数传递页面信息
        router: 'funcA',
    },
}
// context为调用方UIAbility的AbilityContext
this.context.startAbility(wantInfo).then(() => {
    // ...
}).catch((err) => {
    // ...
})
```

情况一：B首次启动时

```ts
import UIAbility from '@ohos.app.ability.UIAbility'
import Window from '@ohos.window'

export default class FuncAbility extends UIAbility {
    onCreate(want, launchParam) {
        // 接收调用方UIAbility传过来的参数
        this.funcAbilityWant = want;
    }

    onWindowStageCreate(windowStage: Window.WindowStage) {
        // Main window is created, set main page for this ability
        let url = 'pages/Index';
        if (this.funcAbilityWant?.parameters?.router === 'funA') {
            url = 'pages/Second';
        }
        windowStage.loadContent(url, (err, data) => {
            // ...
        });
    }
}
```

情况二：B非首次启动

需要使用 onNewWant()回调（当前UIAbility实例之前已经创建完成，不会进入onCreate()和onWindowStageCreate()生命周期回调）

可以通过全局变量globalThis传递参数

```ts
import UIAbility from '@ohos.app.ability.UIAbility'

export default class FuncAbility extends UIAbility {
    onNewWant(want, launchParam) {
        // 接收调用方UIAbility传过来的参数
        globalThis.funcAbilityWant = want;
        // ...
    }
}
```

在B的Index页面中添加逻辑处理

```ts
import router from '@ohos.router';

@Entry
@Component
struct Index {
  onPageShow() {
    let funcAbilityWant = globalThis.funcAbilityWant;
    let url2 = funcAbilityWant?.parameters?.router;
    if (url2 && url2 === 'funcA') {
      router.replaceUrl({
        url: 'pages/Second',
      })
    }
  }
}
```

> 当被调用方Ability的启动模式设置为standard启动模式时，每次启动都会创建一个新的实例，那么onNewWant()回调就不会被用到。

#### 6. 通过Call调用实现UIAbility交互

仅对系统应用开放
