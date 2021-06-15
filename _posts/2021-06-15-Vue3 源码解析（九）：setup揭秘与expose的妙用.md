---
layout: post
title: Vue3 源码解析（九）：setup 揭秘与 expose 的妙用
categories: [前端, Vue]
description: 带你解读 Vue3 源码，setup, 组件
keywords: Vue3, Vue源码, setup, 组件

---

在前几篇文章中我们一起学习了 Vue3 中新颖的 Composition API，而今天笔者要带大家一起看一下 Vue3 中的另一个新鲜的写法 —— setup。

在绝大多数情况，我们书写的组件都是有状态的组件，而这类组件在初始化的过程中会被标记为 stateful comonents，当 Vue3 检测到我们在处理这类有状态组件时，就会调用函数 setupStatefulComponent ，来初始化一个状态化组件。处理组件部分的源码位置在: `@vue/runtime-core/src/component.ts` 。

## setupStatefulComponent

接下来笔者就带着大家一起来剖析一下 setupStatefulComponent 的过程：

```ts
function setupStatefulComponent(
  instance: ComponentInternalInstance,
  isSSR: boolean
) {
  const Component = instance.type as ComponentOptions

  if (__DEV__) { /* 检测组件名称、指令、编译选项等等，有错误则报警 */ }
    
  // 0. 创建一个渲染代理的属性的访问缓存
  instance.accessCache = Object.create(null)
  // 1. 创建一个公共的示例或渲染器代理
  // 它将被标记为 raw，所以它不会被追踪
  instance.proxy = new Proxy(instance.ctx, PublicInstanceProxyHandlers)

  // 2. 调用 setup()
  const { setup } = Component
  if (setup) {
    const setupContext = (instance.setupContext =
      setup.length > 1 ? createSetupContext(instance) : null)

    currentInstance = instance
    pauseTracking()
    const setupResult = callWithErrorHandling(
      setup,
      instance,
      ErrorCodes.SETUP_FUNCTION,
      [__DEV__ ? shallowReadonly(instance.props) : instance.props, setupContext]
    )
    resetTracking()
    currentInstance = null

    if (isPromise(setupResult)) {
      if (isSSR) {
        // 返回一个 promise，因此服务端渲染可以等待它执行。
        return setupResult
          .then((resolvedResult: unknown) => {
            handleSetupResult(instance, resolvedResult, isSSR)
          })
          .catch(e => {
            handleError(e, instance, ErrorCodes.SETUP_FUNCTION)
          })
      }
    } else {
      // 捕获 Setup 执行结果
      handleSetupResult(instance, setupResult, isSSR)
    }
  } else {
    // 完成组件初始化
    finishComponentSetup(instance, isSSR)
  }
}
```

组件一开始会初始化一个 Component 变量，其中保存着组件的选项。接下来如果是 DEV 环境，则会开始检测组件中的各种选项的命名，比如 name、components、directives 等，如果检测有问题，就会在开发环境报出警告。

在检测完毕后，会开始正经的初始化过程，首先会在实例上创建一个 accessCache 的属性，该属性用以缓存渲染器代理属性，以减少读取次数。之后会在组件实例上初始化一个代理属性，这个代理属性代理了组件的上下文，并且将它设置为观察原始值，这样这个代理对象将不会被追踪。

之后就开始处理我们本文关心的 setup 逻辑了。首先从组件中取出 setup 函数，这里判断是否存在 setup 函数，如果不存在，则直接跳转到底部逻辑，执行 finishComponentSetup，完成组件初始化。否则就会进入 `if (setup)` 之后的分支条件中。

是否执行 createSetupContext 生成 setup 的上下文对象，取决于 setup 函数中形参的数量是否大于 1。

这里需要注意的一个知识点是：在 function 函数对象上调用 length 时，返回值是这个函数的形参数量。

举个例子：

```ts
setup() // setup.length === 0

setup(props) // setup.length === 1

setup(props, { emit, attrs }) // setup.length === 2
```

默认情况下，props 是调用 setup 时必传的参数，所以是否需要去生成 setup 的上下文的条件就是 setup.length > 1 。 

那么顺着代码逻辑，我们一起来看一下 setup 上下文中究竟有些什么东西。

```ts
export function createSetupContext(
  instance: ComponentInternalInstance
): SetupContext {
  const expose: SetupContext['expose'] = exposed => {
    instance.exposed = proxyRefs(exposed)
  }

  if (__DEV__) {
    /* DEV 逻辑忽略，对上下文选项设置 getter */
  } else {
    return {
      attrs: instance.attrs,
      slots: instance.slots,
      emit: instance.emit,
      expose
    }
  }
}
```

## expose 的妙用

看到这段 createSetupContext 函数的逻辑，我们发现 setup 上下文中就如文档中描述的一样，有 attrs、slots、emit 这三种熟悉的属性，而在这里惊奇的发现竟然还有一个文档中未说明的 expose 属性返回。

expose 是早先 Vue RFC 中的一个提案，expose 的设想是提供一个像 `expose({ ...publicMembers })` 这样的组合式 API，这样组件的作者就可以在 setup() 中使用该 API 来清除地控制哪些内容会明确地公开暴露给组件使用者。

当你在封装组件时，如果嫌 ref 中暴露的内容过多，不妨用 expose 来约束一下输出。当然这还仅仅是一个 RFC 提案，感兴趣的小伙伴可以偷偷尝鲜哦。

```ts
import { ref } from 'vue'
export default {
  setup(_, { expose }) {
    const count = ref(0)

    function increment() {
      count.value++
    }
    
    // 仅仅暴露 increment 给父组件
    expose({
      increment
    })

    return { increment, count }
  }
}
```

例如当你像上方代码一样使用 expose 时，父组件获取的 ref 对象里只会有 increment 属性，而 count 属性将不会暴露出去。

## 执行 setup 函数

在处理完 setupContext 的上下文后，组件会停止依赖收集，并且开始执行 setup 函数。

```ts
const setupResult = callWithErrorHandling(
  setup,
  instance,
  ErrorCodes.SETUP_FUNCTION,
  [__DEV__ ? shallowReadonly(instance.props) : instance.props, setupContext]
)
```

Vue 会通过 callWithErrorHandling 调用 setup 函数，这里我们可以看最后一行，是作为 args 参数传入的，与上文描述一样，props 会始终传入，若是 setup.length <= 1 , setupContext 则为 null。

调用完 setup 之后，会重置依赖收集状态。接下来判断 setupResult 的返回值类型。

如果 setup 函数的返回值是 promise 类型，并且是服务端渲染的，则会等待继续执行。否则就会报错，说当前版本的 Vue 并不支持 setup 返回 promise 对象。

如果不是 promise 类型返回值，则会通过 handleSetupResult 函数来处理返回结果。

```ts
export function handleSetupResult(
  instance: ComponentInternalInstance,
  setupResult: unknown,
  isSSR: boolean
) {
  if (isFunction(setupResult)) {
    // setup 返回了一个行内渲染函数
    if (__NODE_JS__ && (instance.type as ComponentOptions).__ssrInlineRender) {
      // 当这个函数的名字是 ssrRender (通过 SFC 的行内模式编译)
      // 将函数作为服务端渲染函数
      instance.ssrRender = setupResult
    } else {
      // 否则将函数作为渲染函数
      instance.render = setupResult as InternalRenderFunction
    }
  } else if (isObject(setupResult)) {
    // 将返回对象转换为响应式对象，并设置为实例的 setupState 属性
    instance.setupState = proxyRefs(setupResult)
  }
  finishComponentSetup(instance, isSSR)
}
```

在 handleSetupResult 这个结果捕获函数中，首先判断 setup 返回结果的类型，如果是一个函数，并且又是服务端的行内模式渲染函数，则将该结果作为 ssrRender 属性；而在非服务端渲染的情况下，会直接当做 render 函数来处理。

接着会判断 setup 返回结果如果是对象，就会将这个对象转换成一个代理对象，并设置为组件实例的 setupState 属性。

最终还是会跟其他没有 setup 函数的组件一样，调用 finishComponentSetup 完成组件的创建。

## finishComponentSetup

这个函数的主要作用是获取并为组件设置渲染函数，对于模板（template）以及渲染函数的获取方式有以下三种规范行为：

1、渲染函数可能已经存在，通过 setup 返回了结果。例如我们在上一节讲的 setup 的返回值为函数的情况。

2、如果 setup 没有返回，则尝试获取组件模板并编译，从 `Component.render` 中获取渲染函数，

3、如果这个函数还是没有渲染函数，则将 `instance.render` 设置为空，以便它能从 mixins/extend 等方式中获取渲染函数。

这个在这种规范行为的指导下，首先判断了服务端渲染的情况，接着判断没有 instance.render 存在的情况，当进行这种判断时已经说明组件并没有从 setup 中获得渲染函数，在进行第二种行为的尝试。从组件中获取模板，设置好编译选项后调用 `Component.render = compile(template, finalCompilerOptions)` 进行编译，这部分编译的知识在我的[第一篇文章编译流程](https://originalix.github.io/2021/04/18/Vue3-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90(%E4%B8%80)-%E7%BC%96%E8%AF%91%E6%B5%81%E7%A8%8B/)中有过详细介绍。

最后将编译后的渲染函数赋值给组件实例的 render 属性，如果没有则赋值为 NOOP 空函数。

接着判断渲染函数是否是使用了 with 块包裹的运行时编译的渲染函数，如果是这种情况则会将渲染代理设置为一个不同的 `has` handler 代理陷阱，它的性能更强并且能够去避免检测一些全局变量。

至此组件的初始化完毕，渲染函数也设置结束了。

```ts
export function finishComponentSetup(
  instance: ComponentInternalInstance,
  isSSR: boolean,
  skipOptions?: boolean
) {
  const Component = instance.type as ComponentOptions

  // 模板 / 渲染函数的规范行为
  // 1、渲染函数可能已经存在，通过 setup 返回
  // 2、除此之外尝试使用 `Component.render` 当做渲染函数
  // 3、如果这个函数没有渲染函数，设置 `instance.render` 为空函数，以便它能从 mixins/extend 中获得渲染函数
  if (__NODE_JS__ && isSSR) {
    instance.render = (instance.render ||
      Component.render ||
      NOOP) as InternalRenderFunction
  } else if (!instance.render) {
    // 可以在 setup() 中设置
    if (compile && !Component.render) {
      const template = Component.template
      if (template) {
        const { isCustomElement, compilerOptions } = instance.appContext.config
        const {
          delimiters,
          compilerOptions: componentCompilerOptions
        } = Component
        const finalCompilerOptions: CompilerOptions = extend(
          extend(
            {
              isCustomElement,
              delimiters
            },
            compilerOptions
          ),
          componentCompilerOptions
        )
        Component.render = compile(template, finalCompilerOptions)
      }
    }

    instance.render = (Component.render || NOOP) as InternalRenderFunction

    // 对于使用 `with` 块的运行时编译的渲染函数，这个渲染代理需要不一样的 `has` handler 陷阱，它有更好的
    // 性能表现并且只允许白名单内的 globals 属性通过。
    if (instance.render._rc) {
      instance.withProxy = new Proxy(
        instance.ctx,
        RuntimeCompiledPublicInstanceProxyHandlers
      )
    }
  }
}

```

## 总结

今天笔者介绍了一个有状态的组件的初始化的过程，在 setup 函数初始化部分进行了仔细的讲解，我们不仅学习了 setup 上下文初始化的条件，也明确的知晓了 setup 上下文究竟给我们暴露了哪些属性，并且从中学到了一个新的 RFC 提案： expose 属性。

我们学习了 setup 函数执行的过程以及 Vue 是如何处理捕获 setup 的返回结果的。

最后我们讲解了组件初始化时，不论是否使用 setup 都会执行的 finishComponentSetup 函数，通过这个函数内部的逻辑我们了解了一个组件在初始化完毕时，渲染函数设置的规则。

最后，如果这篇文章能够帮助到你了解 Vue3 中 setup 的小细节，希望能给本文点一个喜欢❤️。如果想继续追踪后续文章，也可以关注我的账号或 follow 我的 [github](https://github.com/originalix)，再次谢谢各位可爱的看官老爷。

