---
layout: post
title: Vue3 源码解析（八）：ref 与 computed 原理揭秘
categories: [前端, Vue]
description: 带你解读 Vue3 源码，ref, computed, 计算属性, composition api
keywords: Vue3, Vue源码, 响应式原理, ref, computed, 计算属性

---

在 Vue3 新推出的响应式 API 中，Ref 系列毫无疑问是使用频率最高的 api 之一，而 computed 计算属性是一个在上一个版本中就非常熟悉的选项了，但是在 Vue3 中也提供了独立的 api 方便我们直接创建计算值。而今天这篇文章，笔者就会给大家讲解 ref 与 computed 的实现原理，让我们一起开始本章的学习吧。

## ref

当我们有一个独立的原始值，例如一个字符串，我们想让它变成响应式的时候可以通过创建一个对象，将这个字符串以键值对的形式放入对象中，然后传递给 reactive。而 Vue 为我们提供了一个更容易的方式，通过 ref 来完成。

```ts
import { ref } from 'vue'
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

ref 会返回一个可变的响应式对象，该对象作为一个响应式的引用维护着它内部的值，这就是 ref 名称的来源。该对象只包含一个名为 value 的 property。

而 ref 究竟是如何实现的呢？

ref 的源码位置在 @vue/reactivity 的库内，路径是 packages/reactivity/src/ref.ts ，接下来我们就一起来看 ref 的实现。

```ts
export function ref<T extends object>(value: T): ToRef<T>
export function ref<T>(value: T): Ref<UnwrapRef<T>>
export function ref<T = any>(): Ref<T | undefined>
export function ref(value?: unknown) {
  return createRef(value)
}
```

从 ref api 的函数签名中，可以看到 ref 函数接收一个任意类型的值作为它的 value 参数，并返回一个 Ref 类型的值。

```ts
export interface Ref<T = any> {
  value: T
  [RefSymbol]: true
  _shallow?: boolean
}
```

从返回值 Ref 的类型定义中看出，ref 的返回值中有一个 value 属性，以及有一个私有的 symbol key，还有一个标识是否为 shallowRef 的_shallow 布尔类型的属性。

函数体内直接返回了 createRef 函数的返回值。

### createRef

```ts
function createRef(rawValue: unknown, shallow = false) {
  if (isRef(rawValue)) {
    return rawValue
  }
  return new RefImpl(rawValue, shallow)
}
```

createRef 的实现也很简单，入参为 rawValue 与 shallow，rawValue 记录的创建 ref 的原始值，而 shallow 则是表明是否为 shallowRef 的浅层响应式 api。

函数的逻辑为先使用 isRef 判断是否为 rawValue，如果是的话则直接返回这个 ref 对象。

否则返回一个新创建的 RefImpl 类的实例对象。

### RefImpl 类

```ts
class RefImpl<T> {
  private _value: T

  public readonly __v_isRef = true

  constructor(private _rawValue: T, public readonly _shallow: boolean) {
    // 如果是 shallow 浅层响应，则直接将 _value 置为 _rawValue，否则通过 convert 处理 _rawValue
    this._value = _shallow ? _rawValue : convert(_rawValue)
  }

  get value() {
    // 读取 value 前，先通过 track 收集 value 依赖
    track(toRaw(this), TrackOpTypes.GET, 'value')
    return this._value
  }

  set value(newVal) {
    // 如果需要更新
    if (hasChanged(toRaw(newVal), this._rawValue)) {
      // 更新 _rawValue 与 _value
      this._rawValue = newVal
      this._value = this._shallow ? newVal : convert(newVal)
      // 通过 trigger 派发 value 更新
      trigger(toRaw(this), TriggerOpTypes.SET, 'value', newVal)
    }
  }
}
```

在 RefImpl 类中，有一个私有变量 _value 用来存储 ref 的最新的值；公共的只读变量 __v_isRef 是用来标识该对象是一个 ref 响应式对象的标记与在讲述 reactive api 时的 ReactiveFlag 相同。

而在 RefImpl 的构造函数中，接受一个私有的 _rawValue 变量，存放 ref 的旧值；公共的 _shallow 变量是区分是否为浅层响应的。在构造函数内部，先判断 _shallow 是否为 true，如果是 shallowRef ，则直接将原始值赋值给 _value，否则会通过 convert 进行转换再赋值。

在 conver 函数的内部，其实就是判断传入的参数是否是一个对象，如果是一个对象则通过 reactive api 创建一个代理对象并返回，否则直接返回原参数。

当我们通过 ref.value 的形式读取该 ref 的值时，就会触发 value 的 getter 方法，在 getter 中会先通过 track 收集该 ref 对象的 value 的依赖，收集完毕后返回该 ref 的值。

当我们对 ref.value 进行修改时，又会触发 value 的 setter 方法，会将新旧 value 进行比较，如果值不同需要更新，则先更新新旧 value，之后通过 trigger 派发该 ref 对象的 value 属性的更新，让依赖该 ref 的副作用函数执行更新。

如果有朋友对于 track 收集依赖，trigger 派发更新比较迷糊的话，建议先阅读我的[上一篇文章](https://originalix.github.io/2021/06/10/Vue3-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90-%E4%B8%83-%E4%BE%9D%E8%B5%96%E6%94%B6%E9%9B%86%E4%B8%8E%E5%89%AF%E4%BD%9C%E7%94%A8%E5%87%BD%E6%95%B0/)，在上一篇文章中笔者仔细讲解了这个过程，至此 ref 的实现笔者就给大家解释清楚了。

## computed

在文档中关于 computed api 是这样介绍的：接受一个 getter 函数，并以 getter 函数的返回值返回一个不可变的响应式 ref 对象。或者它也可以使用具有 get 和 set 函数的对象来创建一个可写的 ref 对象。

### computed 函数

根据这个 api 的描述，显而易见的能够知道 computed 接受一个函数或是对象类型的参数，所以我们先从它的函数签名看起。

```ts
export function computed<T>(getter: ComputedGetter<T>): ComputedRef<T>
export function computed<T>(
  options: WritableComputedOptions<T>
): WritableComputedRef<T>
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>
)
```

在 computed 函数的重载中，代码第一行接收 getter 类型的参数，并返回 ComputedRef 类型的函数签名是文档中描述的第一种情况，接受 getter 函数，并以 getter 函数的返回值返回一个不可变的响应式 ref 对象。

而在第二行代码中，computed 函数接受一个 options 对象，并返回一个可写的 ComputedRef 类型，是文档的第二种情况，创建一个可写的 ref 对象。

第三行代码，则是这个函数重载的最宽泛情况，参数名已经提现了这一点：getterOrOptions。

一起看一下 computed api 中相关的类型定义：

```ts
export interface ComputedRef<T = any> extends WritableComputedRef<T> {
  readonly value: T
}

export interface WritableComputedRef<T> extends Ref<T> {
  readonly effect: ReactiveEffect<T>
}

export type ComputedGetter<T> = (ctx?: any) => T
export type ComputedSetter<T> = (v: T) => void

export interface WritableComputedOptions<T> {
  get: ComputedGetter<T>
  set: ComputedSetter<T>
}
```

从类型定义中得知：WritableComputedRef 以及 ComputedRef 都是扩展自 Ref 类型的，这也就理解了文档中为什么说 computed 返回的是一个 ref 类型的响应式对象。

接下来看一下 computed api 的函数体内的完整逻辑：

```ts
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>
) {
  let getter: ComputedGetter<T>
  let setter: ComputedSetter<T>

  // 如果 参数 getterOrOptions 是一个函数
  if (isFunction(getterOrOptions)) {
   	// 那么这个函数必然就是 getter，将函数赋值给 getter
    getter = getterOrOptions
    // 这种场景下如果在 DEV 环境下访问 setter 则报出警告
    setter = __DEV__
      ? () => {
          console.warn('Write operation failed: computed value is readonly')
        }
      : NOOP
  } else {
    // 这个判断里，说明参数是一个 options，则取 get、set 赋值即可
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }
  
  return new ComputedRefImpl(
    getter,
    setter,
    isFunction(getterOrOptions) || !getterOrOptions.set
  ) as any
}
```

在 computed api 中，首先会判断传入的参数是一个 getter 函数还是 options 对象，如果是函数的话则这个函数只能是 getter 函数无疑，此时将 getter 赋值，并且在 DEV 环境中访问 setter 不会成功，同时会报出警告。如果传入是不是函数，computed 就会将它作为一个带有 get、set 属性的对象处理，将对象中的 get、set 赋值给对应的 getter、setter。最后在处理完成后，会返回一个 ComputedRefImpl 类的实例对象，computed api 就处理完成。

### ComputedRefImpl 类

这个类与我们之前介绍的 RefImpl Class 类似，但构造函数中的逻辑有点区别。

先看类中的成员变量：

```ts
class ComputedRefImpl<T> {
  private _value!: T
  private _dirty = true

  public readonly effect: ReactiveEffect<T>

  public readonly __v_isRef = true;
  public readonly [ReactiveFlags.IS_READONLY]: boolean
}
```

跟 RefImpl 类相比，增加了 _dirty 私有成员变量，一个 effect 的只读副作用函数变量，以及增加了一个 __v_isReadonly 标记。

接着看一下构造函数中的逻辑：

```ts
constructor(
  getter: ComputedGetter<T>,
  private readonly _setter: ComputedSetter<T>,
  isReadonly: boolean
) {
  this.effect = effect(getter, {
    lazy: true,
    scheduler: () => {
      if (!this._dirty) {
        this._dirty = true
        trigger(toRaw(this), TriggerOpTypes.SET, 'value')
      }
    }
  })

  this[ReactiveFlags.IS_READONLY] = isReadonly
}
```

构造函数中，会为 getter 创建一个副作用函数，并且在副作用选项中设置为延迟执行，并且增加了调度器。在调度器中会判断 this._dirty 标记是否为 false，如果是的话，将 this._dirty 置为 true，并且利用 trigger 派发更新。如果对这个副作用的执行时机，以及副作用中调度器是什么时候执行这些问题犯迷糊的同学，还是建议阅读[上一篇文章](https://originalix.github.io/2021/06/10/Vue3-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90-%E4%B8%83-%E4%BE%9D%E8%B5%96%E6%94%B6%E9%9B%86%E4%B8%8E%E5%89%AF%E4%BD%9C%E7%94%A8%E5%87%BD%E6%95%B0/)，先把 effect 副作用搞明白，再去理解响应式的其他 api 必然是事半功倍的。

```ts
get value() {
  // 这个 computed ref 有可能是被其他代理对象包裹的
  const self = toRaw(this)
  if (self._dirty) {
    // getter 时执行副作用函数，派发更新，这样能更新依赖的值
    self._value = this.effect()
    self._dirty = false
  }
  // 调用 track 收集依赖
  track(self, TrackOpTypes.GET, 'value')
  // 返回最新的值
  return self._value
}

set value(newValue: T) {
  // 执行 setter 函数
  this._setter(newValue)
}
```

在 computed 中，通过 getter 函数获取值时，会先执行副作用函数，并将副作用函数的返回值赋值给 _value，并将 _dirty 的值赋值给 false，这就可以保证如果 computed 中的依赖没有发生变化，则副作用函数不会再次执行，那么在 getter 时获取到的 _dirty 始终是 false，也不需要再次执行副作用函数，节约开销。之后通过 track 收集依赖，并返回 _value 的值。

而在 setter 中，只是执行我们传入的 setter 逻辑，至此 computed api 的实现也已经讲解完毕了。

## 总结

在本文中，以上文副作用函数和依赖收集派发更新的知识点为基础，笔者为大家讲解了 ref 和 computed 两个在 Vue3 响应式中最常用的 api 的实现，这两个 api 都是在创建时返回了一个类实例，在实例中的构造函数以及对 value 属性设置的 get 和 set 完成响应式追踪。

当我们在学会使用这些的同时，并能知其所以然一定能够帮我们在使用这些 api 时发挥出它最大的作用，同时也能够让你在写出了一些不符合你预期代码的时候，快速的定位问题，能搞定究竟是自己写的不对，还是本身 api 并不支持某种调用方式。

最后，如果这篇文章能够帮助到你了解 Vue3 中的响应式 api ref 和 computed 的实现原理，希望能给本文点一个喜欢❤️。如果想继续追踪后续文章，也可以关注我的账号或 follow 我的 [github](https://github.com/originalix)，再次谢谢各位可爱的看官老爷。
