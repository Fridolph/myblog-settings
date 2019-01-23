---
title: 【Vue源码】数据驱动(1)
date: 2019-1-16 22:19:23
tags:
  - vue
  - 源码
  - 数据驱动
categories:
  - vue源码学习
---

> 新年新气象，加油 2019！ 其实这是去年就该做的事，因为太懒，于是现在补来也不晚，正好坐等 Vue 3 的大更新。迈开这一小步也是今后进阶的一大步，加油！那么就进入正题了

![Vue数据驱动](http://blog.fueson.top/article/img/1Data-driven.jpg)

<!-- more -->

# 数据驱动

Vue 一个核心思想是数据驱动。所谓数据驱动，是指视图是由数据驱动生成的，我们对视图的修改，不会直接操作 DOM，而是通过修改数据。它相比我们传统的前端开发，如使用 jQuery 等前端库直接修改 DOM，大大简化了代码量。特别是当交互复杂时，只关心数据的修改会让代码的逻辑变得更清晰，因为 DOM 变成数据的映射，我们所有的逻辑都是对数据的修改，而不用碰触 DOM，这样的代码非常利于维护。

## new Vue 时都做了什么

```js
function Vue(options) {
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options);
}
```

Vue 只能通过 new 关键字初始化，然后会调用 `this._init()`方法，

```js
Vue.prototype._init = function(options?: Object) {
  const vm: Component = this;
  // a uid
  vm._uid = uid++;

  let startTag, endTag;
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    startTag = `vue-perf-start:${vm._uid}`;
    endTag = `vue-perf-end:${vm._uid}`;
    mark(startTag);
  }

  // a flag to avoid this being observed
  vm._isVue = true;
  // merge options
  if (options && options._isComponent) {
    // 优化内部组件实例化，因为动态选项合并非常慢，并且没有一个内部组件选项需要特殊处理
    initInternalComponent(vm, options);
  } else {
    vm.$options = mergeOptions(resolveConstructorOptions(vm.constructor), options || {}, vm);
  }
};
```

Vue 初始化主要就干了几件事：

- 合并配置
- 初始化生命周期
- 初始化事件中心
- 初始化渲染
- 初始化 data、props、computed、watcher 等等

### 小结

Vue 的初始化逻辑很清楚，把不同的功能逻辑拆成一个单独的函数执行，主线逻辑一目了然，这样的编程思想非常值得学习和借鉴。

```js
function test() {
  return () => {
    return () => {};
  };
}
```

## Vue 实例挂载

Vue 中通过\$mount 实例方法挂载 vm

$mount方法的实现和平台、构建方式相关。这里分析带compiler版本的$mount 实现

src/platform/web/entry-runtime-with-compiler.js

```js
const mount = Vue.prototype.$mount;
Vue.prototype.$mount = function(el?: string | Element, hydrating?: boolean): Component {
  el = el && query(el);

  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' &&
      warn(`Do not mount Vue to <html> or <body> - mount to normal elements instead.`);
    return this;
  }

  const options = this.$options;
  // 解析模板 / el并转换为渲染功能
  if (!options.render) {
    let template = options.template;
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template);
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(`Template element not found or is empty: ${options.template}`, this);
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML;
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this);
        }
        return this;
      }
    } else if (el) {
      template = getOuterHTML(el);
    }
    if (template) {
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile');
      }
      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          shouldDecodeNewlines,
          shouldDecodeNewlineForHref,
          delimiters: options.delimiters,
          comments: options.comments,
        },
        this,
      );
      options.render = render;
      options.staticRenderFns = staticRenderFns;

      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end');
        measure(`vue ${this._name} compile`, 'compile', 'compile end');
      }
    }
  }
  return mount.call(this, el, hydrating);
};
```

缓存原型上的\$mount 方法，再重定义该方法（call 改变 this 指向为 Vue.prototype）

第一个 if 块是对 el 做限制，Vue 不能挂载在 body、html 这类根节点上。

若没有定义 render 方法，则把 el 或者 template 字符串转换成 render 方法。所有 Vue 组件的渲染最终都需要 render 方法， 无论是单文件还是.vue 方式上写的 el 或是 template，最终都会转换成 render 方法， 这是一个在线编译过程（这是调用 compileToFunctions）方法实现的。最后，再调用原先原型上的\$mount 方法挂载。

---

那之前原型的\$mount 方法在 src/platform/web/runtime/index.js 中定义的，这么拆分设计的理由是为了复用，因为它可以被 runtime only 版本的 Vue 直接使用。

```js
// public mount method
Vue.prototype.$mount = function(el?: string | Element, hydrating?: boolean): Component {
  el = el && inBrowser ? query(el) : undefined;
  return mountComponent(this, el, hydrating);
};
```

\$mount 方法支持传入 2 参数，第 1 个是 el，表挂载元素（字符串或 DOM），字符串在浏览器下调用 query 方法转换成 dom；第 2 个参数是和服务端渲染相关，在浏览器环境下我们不需要传第二个参数

\$mount 方法实际上会调用 mountComponent 方法，这个方法定义在 src/core/instance/lifecycle.js 中：

```js
export function mountComponent(vm: Component, el?: Element, hydrating?: boolean): Component {
  vm.$el = el;
  if (!vm.$options.render) {
    vm.$options.render = createEmptyNode;
    if (process.env.NODE_ENV !== 'production') {
      if (
        (vm.$options.template && vm.$options.template.chatAt(0) !== '#') ||
        vm.$options.el ||
        el
      ) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
            'compiler is not available. Either pre-compile the templates into ' +
            'render functions, or use the compiler-included build.',
          vm,
        );
      } else {
        warn('Failed to mount component: template or render function not defined.', vm);
      }
    }
  }
  callHook(vm, 'beforeMount');

  let updateComponent;
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name;
      const id = vm._uid;
      const startTag = `vue-perf-start:${id}`;
      const endTag = `vue-perf-end:${id}`;

      mark(startTag);
      const vnode = vm._render();
      mark(endTag);
      measure(`vue ${name} render`, startTag, endTag);

      mark(startTag);
      vm._update(vnode, hydrating);
      mark(endTag);
      measure(`vue ${name} patch`, startTag, endTag);
    };
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating);
    };
  }

  // 我们在观察者的构造函数中将其设置为 vm._watcher，
  // 因为观察者的初始补丁可能会调用 $forceUpdate（例如，在子组件的挂载挂钩内），这依赖于已定义的vm._watcher
  new Watcher(
    vm,
    updateComponent,
    noop,
    {
      before() {
        if (vm._isMounted) {
          callHook(vm, `beforeUpdate`);
        }
      },
    },
    true /* isRenderWatcher */,
  );
  hydrating = false;
  // 手动挂载实例，调用挂载在自己身上
  // 在插入的钩子中调用了渲染创建的子组件
  if (vm.$vnode == null) {
    vm._isMounted = true;
    callHook(vm, 'mounted');
  }
  return vm;
}
```

从上， `mountComponent` 核心就是先调用 vm.\_render 方法先生成虚拟 Node, 再实例化一个渲染 Watcher, 在它的回调中调用 updateComponent 方法，最终调用 vm.\_update 更新 dom

Watcher 的作用有二：

1. 初始化时执行回调
2. 当 vm 实例的检测数据变化时执行回调

函数判断根节点时设置 `vm._isMounted = true` 表示实例已挂载，同时执行 mounted 钩子。这里 vm.\$vnode 表示 Vue 实例的父虚拟 Node，所以它为 null 表示当前是根 Vue 实例

### 小结

mountComponent 方法的逻辑很清晰，它会完成整个渲染工作 核心就是 vm.\_render 和 vm.\_update

## render

Vue 的 \_render 方法是实例的一个私有方法，它用来把实例渲染成一个虚拟 Node。它的定义在 `src/core/instance/render.js` 文件中：

```js
Vue.prototype._render = function(): VNode {
  const vm: Component = this;
  const { render, _parentVnode } = vm.$options;

  // reset _rendered flag on slots for duplicate slot check
  if (process.env.NODE_ENV !== 'production') {
    for (const key in vm.$slots) {
      // $flow-disabled-line
      vm.$slots[key]._rendered = false;
    }
  }

  if (_parentVnode) {
    vm.$scopedSlots = _parentVnode.data.scopedSlots || emptyObject;
  }

  // set parent vnode. this allows render functions to have access
  // to the data on the placehoder node.
  vm.$vnode = _parentVnode;
  // render self
  let vnode;
  try {
    vnode = render.call(vm._renderProxy, vm.$createElement);
  } catch (e) {
    handleError(e, vm, `render`);
    // return error render result,
    // or previous vnode to prevent render error causing blank component
  }
};
```

这段代码最关键的是 `render` 方法的调用，我们在平时的开发工作中手写 render 方法的场景比较少，而写的比较多的是 template 模版，在之前的 mounted 方法的实现中，会把 template 编译成 render 方法，但这个编译过程很复杂（这里暂略过）。

在 Vue 的官方文档中介绍了 render 函数的第一个参数是`createElement`，那么结合之前的例子：

```html
<div id="app">{{message}}</div>
```

相当于我们编写了如下 render 函数：

```js
render: function(createElement) {
  return createElement('div', {
    attrs: {
      id: 'app'
    },
  }, this.message)
}
```

再回到 \_render 函数中的 render 方法的调用：

```js
vnode = render.call(vm._renderProxy, vm.$createElement);
```

可以看到，`render`函数中的`createElement`方法就是`vm.$createElement`方法：

```js
export function initRender(vm: Component) {
  // ...
  // bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false);
  // normalization is always applied for the public version, used in
  // user-written render functions
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true);
}
```

实际上，vm.\$createElement 方法定义是在执行`initRender`方法时，可以看到除了`vm.$createElement`方法，还有一个 `vm.c` 方法，它是被模版编译成的 render 函数使用，而 vm.\$createElement 是用户手写 render 方法使用的，这两个方法支持的参数相同，并且内部都调用了 createElement 方法。

### 小结

vm.\_render 最终是通过执行 createElement 方法并返回的是 vnode，它是一个虚拟 Node. Vue2 版本相比之前最大的提升就是利用了 Virtual DOM。因此在分析 createElement 的实现前，我们先了解一下 Virtual DOM 的概念。

## update

Vue 的 \_update 是实例的一个私有方法，它被调用的时机有 2 个，一个是首次渲染，一个是数据更新时。

\_update 方法的作用是把 VNode 渲染成真实的 DOM，它的定义在 src/core/instance/lifecycle.js

```js
Vue.prototype._update = function(vnode: Vnode, hydrating?: boolean) {
  const vm: Component = this;
  const prevEl = vm.$el;
  const prevVnode = vm._vnode;
  const prevActiveInstance = activeInstance;
  activeInstance = vm;
  vm._vnode = vnode;
  // Vue.ptototype.__patch__ is injected in entry points
  // based on the rendering backend used.
  if (!prevVnode) {
    // initial render
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false);
  } else {
    // updates
    vm.$el = vm.__patch__(prevVnode, vnode);
  }
  activeInstance = prevActiveInstance;
  // udpate __vue__ reference
  if (prevEl) {
    prevEl.__vue__ = null;
  }
  if (vm.$el) {
    vm.$el.__vue__ = vm;
  }
  // if parent is an HOC , update its $el as well
  if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
    vm.$parent.$el = vm.$el;
  }
  // update hook is called by the scheduler to ensure that children are
  // updated in a parent's updated hook.
};
```

\_update 的核心就是调用 vm.**patch** 方法，这个方法实际上在不同的平台，比如 web 和 weex 上的定义就是不一样的，因此在 web 平台中它的定义在 src/platforms/web/runtime/index.js 中：

```js
Vue.prototype.__patch__ = inBrowser ? patch : noop;
```

可以看到，甚至在 web 平台上，是否是服务端渲染也会对该方法产生影响。 因为在服务端渲染中，没有真实的浏览器 DOM 环境，所以不需要把 VNode 最终转换成 DOM，因此是一个空函数，而在浏览器端渲染中，它指向了 patch 方法，它的定义在 src/platforms/web/runtime/patch.js 中：

```js
import * as nodeOps from 'web/runtime/node-ops';
import { createPatchFunction } from 'core/vdom/patch';
import baseModules from 'core/vdom/modules/index';
import platformModules from 'web/runtime/modules/index';

// the directive module should be applied last, after all
// built-in modules have been applied.
const modules = platformModules.concat(baseModules);

export const patch: Function = createPatchFunction({ nodeOps, modules });
```

该方法的定义是调用 createPatchFunction 方法的返回值，这里传入了一个对象，包含 nodeOps 参数和 modules 参数。其中，nodeOpts 封装了一系列 DOM 操作方法，modules 定义了一些模块的钩子函数的实现，先来看一下 createPatchFunction 的实现， 它定义在 src/core/vdom/patch.js 中：

```js
const hooks = ['create', 'activate', 'update', 'remove', 'destroy'];

export function createPatchFunction(backend) {
  let i, j;
  const cbs = {};
  const { modules, nodeOps } = backend;

  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = [];
    for (j = 0; j < modules.length; ++j) {
      if (isDef(modules[j][hooks[i]])) {
        cbs[hooks[i]].push(modules[j][hooks[i]]);
      }
    }
  }

  return function patch(oldVnode, vnode, hydrating, removeOnly) {
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode);
      return;
    }

    let isInitialPatch = false;
    const insertedVnodeQueue = [];

    if (isUndef(oldVnode)) {
      // empty mount (likely as component), create new root element
      isInitialPatch = true;
      createElm(vnode, insertedVnodeQueue);
    } else {
      const isRealElement = isDef(oldVnode, nodeType);
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly);
      } else {
        if (isRealElement) {
          // mounting to a real element
          // check if this is server-rendered countent
          // and if we can perform a successful hydration.
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            oldVnode.removeAttributes(SSR_ATTR);
            hydrating = true;
          }
          if (isTrue(hydrating)) {
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              invokeInsertHook(vnode, insertedVnodeQueue, true);
              return oldVnode;
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                  'server-rendered content. This is likely caused by incorrect ' +
                  'HTML markup, for example nesting block-level elements inside ' +
                  '<p>, or missing <tbody>. Bailing hydration and performing ' +
                  'full client-side render.',
              );
            }
          }
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          oldVnode = emptyNodeAt(oldVnode);
        }
        // replacing existing element
        const oldElm = oldVnode.elm;
        const parentElm = nodeOps.parentNode(oldElm);
        // create new node
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm),
        );
        // update parent placeholder node element, recursively
        if (isDef(vnode.parent)) {
          let ancestor = vnode.parent;
          const patchable = isPatchable(vnode);
          while (ancestor) {
            for (let i = 0; i < cbs.destroy.length; i++) {
              cbs.destroy[i](ancestor);
            }
            ancestor.elm = vnode.elm;
            if (patchable) {
              for (let i = 0; i < cbs.create.length; i++) {
                cbs.create[i](emptyNode, ancestor);
              }
              const insert = ancestor.data.hook.insert;
              if (insert.merged) {
                for (let i = 1; i < insert.fns.length; i++) {
                  insert.fns[i]();
                }
              }
            } else {
              registerRef(ancestor);
            }
            ancestor = ancestor.parent;
          }
        }
        // destroy old node
        if (isDef(parentElm)) {
          removeVnodes(parentElm, [oldVnode], 0, 0);
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode);
        }
      }
    }
    invokeInsertHook(vnode, insertedVnodeQueue, isIntialPatch);
    return vnode.elm;
  };
}
```

createPatchFunction 内部定义了一系列辅助方法，最终返回一个 patch 方法，这个方法就赋值给了 vm.\_update 函数里调用的 vm.**patch**

在介绍 patch 方法实现之前，我们可以思考一下为何 vue.js 源码绕了这么一大圈，把相关代码分散到各个目录。因为前面介绍过，patch 是平台相关的，在 web 和 weex 环境下，它们把虚拟 dom 映射到平台 DOM 的方法是不同的，并且对 DOM 包括的属性模块创建和更新也不尽相同。因此每个平台都有各自的 nodeOps 和 modules，它们的代码需要托管在 src/platforms 这个大目录下

而不同平台的 patch 的主要逻辑部分是相同的，所以这部分公共的部分托管在 core 这个大目录下。差异化部分只需要通过参数来区别，这里用到了一个函数柯里化的技巧，通过 createPatchFunction 把差异参数提前固化，这样就不用每次调用 patch 时都传递 nodeOps 和 modules 了。这种编程技巧也值得学习。

在这里，nodeOps 表示对平台 DOM 的一些操作方法， modules 表示平台的一些模块，它们会在整个 patch 过程的不同阶段执行相应的钩子函数。这些代码的具体实现会在后面的章节介绍。

回到 patch 方法本身，它接受 4 个参数：

- oldVnode 表示旧的 vnode 节点，它也可以不存在或是一个 DOM 对象；
- vnode 表示执行 \_render 返回后的 Vnode 节点；
- hydrating 表示是否服务端渲染；
- removeOnly 是给 transition-group 用的

`patch` 逻辑看上去相对复杂，因为它有着非常多的分支逻辑，为方便理解，这里仅针对我们之前的例子分析它的执行逻辑。

先回顾一下我们的例子

```js
var app = new Vue({
  el: '#app',
  render(createElement) {
    return createElement(
      'div',
      {
        attrs: {
          id: 'app',
        },
      },
      this.message,
    );
  },
  data: {
    message: 'hello vue!',
  },
});
```

然后我们在 vm.\_update 方法里是这么调用 patch 方法的：

```js
vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false);
```

结合我们的例子，我们的场景是首次渲染，所以在执行 patch 函数时，传入的 vm.$el 对应的是 例子中的id为 app的DOM对象，这个也就是我们在index.html模版中写的 `<div id="app">` vm.$el 的赋值是在之前 mountComponent 函数做的，vnode 对应的是调用 render 函数的返回值，hydrating 是在非服务端渲染情况下为 false, removeOnly 为 false。确定这些传参后，我们回到 patch 函数的执行过程， 看几个关键步骤：

```js
const isRealElement = isDef(oldVnode.nodeType);
if (!isRealElement && sameVnode(oldVnode, vnode)) {
  // patch existing root node
  patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly);
} else {
  if (isRealElement) {
    // mounting to a real element
    // check if this is server-rendered content and if we can perform
    // a successful hydration.
    if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
      oldVnode.removeAttribute(SSR_ATTR);
      hydrating = true;
    }
    if (isTrue(hydrating)) {
      if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
        invokeInsertHook(vnode, insertedVnodeQueue, true);
        return oldVnode;
      } else if (process.env.NODE_ENV !== 'production') {
        warn(
          'The client-side rendered virtual DOM tree is not matching ' +
            'server-rendered content. This is likely caused by incorrect ' +
            'HTML markup, for example nesting block-level elements inside ' +
            '<p>, or missing <tbody>. Bailing hydration and performing ' +
            'full client-side render.',
        );
      }
    }
    // either not server-rendered, or hydration failed.
    // create an empty node and replace it
    oldVnode = emptyNodeAt(oldVnode);
  }

  // replacing existing element
  const oldElm = oldVnode.elm;
  const parentElm = nodeOps.parentNode(oldElm);

  // create new node
  createElm(
    vnode,
    insertedVnodeQueue,
    // extremely rare edge case: do not insert if old element is in a
    // leaving transition. Only happens when combining transition +
    // keep-alive + HOCs. (#4590)
    oldElm._leaveCb ? null : parentElm,
    nodeOps.nextSibling(oldElm),
  );
}
```

由于我们传入的 oldVnode 实际上是一个 DOM container ，所以 isRealElement 为 true，接下来又通过 emptyNodeAt 方法把 oldVnode 转换成 Vnode 对象。然后又调用 createElm 方法，这个方法在这里非常重要，我们来看下它的实现：

```js
function createElm(vnode, insertedVnodeQueue, parentElm, refElm, nested, ownerArray, index) {
  if (isDef(vnode.elm) && isDef(ownerArray)) {
    vnode = ownerArray[index] = cloneVnode(vnode);
  }

  vnode.isRootInsert = !nested;
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return;
  }

  const data = vnode.data;
  const children = vnode.children;
  const tag = vnode.tag;
  if (isDef(tag)) {
    if (process.env.NODE_ENV !== 'production') {
      if (data && data.pre) {
        creatingElmInVPre++;
      }
      if (isUnknownElement(vnode, creatingElmInVPre)) {
        warn(
          'Unknown custom element: <' +
            tag +
            '> - did you ' +
            'register the component correctly? For recursive components, ' +
            'make sure to provide the "name" option.',
          vnode.context,
        );
      }
    }
    vnode.elm = vnode.ns
      ? nodeOps.createElementNS(vnode.ns, tag)
      : nodeOps.createElement(tag, vnode);
    setScope(vnode);

    if (__WEEX__) {
      //
    } else {
      createChildren(vnode, children, insertedVnodeQueue);
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue);
      }
      insert(parentElm, vnode.elm, refElm);
    }
    if (process.env.NODE_ENV !== 'production' && data && data.pre) {
      creatingElmInVPre--;
    }
  } else if (isTrue(vnode.isComment)) {
    vnode.elm = nodeOps.createComment(vnode.text);
    insert(parentElm, vnode.elm, refElm);
  } else {
    vnode.elm = nodeOps.createTextNode(vnode.text);
    insert(parentElm, vnode.elm, refElm);
  }
}
```

createElm 的作用是通过虚拟节点创建真实 DOM 并插入到它的父节点中。我们来看一下关键的逻辑。createComponent 方法目的是尝试创建子节点，这个逻辑在之后组件的章节会详细介绍，在当前这个 case 下它的返回值为 false，接下来判断 vnode 是否包含 tag，如果包含，先简单地对 tag 的合法性在非生产环境做校验，看是否是一个合法的标签；然后再去调用平台的 DOM 操作去创建一个占位符元素。

```js
vnode.elm = vnode.ns ? nodeOps.createElementNS(vnode.ns, tag) : nodeOps.createElement(tag, vnode);
```

接下来，调用 createChildren 方法去创建子元素：

```js
createChildren(vnode, children, insertedVnodeQueue);

function createChildren(vnode, children, insertedVnodeQueue) {
  if (Array.isArray(children)) {
    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(children);
    }
    for (let i = 0; i < children.length; ++i) {
      createElm(children[i], insertedVnodeQueue, vnode.elm, null, true, children, i);
    }
  } else if (isPrimitive(vnode.text)) {
    nodeOps.append;
  }
}
```

createChildren 的逻辑很简单，实际上是比那里子虚拟节点，递归调用 createElm，这是一种常用的深度优先的遍历算法，这里要注意的一点是在遍历过程中会把 vnode.elm 作为父容器的 DOM 节点占位符传入。

接着再调用 invokeCreateHooks 方法执行所有的 create 的钩子并把 vnode push 到 instertedVnodeQueue 中：

```js
if (isDef(data)) {
  invokeCreateHooks(vnode, insertedVnodeQueue);
}

function invokeCreateHooks(vnode, insertedVnodeQueue) {
  for (let i = 0; i < cbs.create.length; ++i) {
    cbs.create[i](emptyNode, vnode);
  }
  i = vnode.data.hook; // Reuse variable
  if (isDef(i)) {
    if (isDef(i.create)) i.create(emptyNode, vnode);
    if (isDef(i.insert)) insertedVnodeQueue.push(vnode);
  }
}
```

最后调用 insert 方法把 dom 插入到父节点中，因为是递归调用，子元素会优先调用 insert，所以整个 vnode 树节点的插入顺序是先子后父。再来看一下 insert 方法，它的定义在 src/core/vdom/patch.js 上：

```js
insert(parentElm, vnode.elm, refElm);

function insert(parent, elm, ref) {
  if (isDef(parent)) {
    if (isDef(ref)) {
      if (ref.parentNode === parent) {
        nodeOps.insertBefore(parent, elm, ref);
      }
    } else {
      nodeOps.appendChild(parent, elm);
    }
  }
}
```

insert 的逻辑简单一些，调用一些 nodeOps 把子节点插入到父节点中，这些辅助方法定义在 src/platforms/web/runtime/node-ops.js 中：

```js
export function insertBefore(parentNode: Node, newNode: Node, refrenceNode: Node) {
  parentNode.insertBefore(newNode, referenceNode);
}

export function appendChild(node: Node, child: Node) {
  node.appendChild(child);
}
```

其实就是调用原生 DOM 的 api 进行 DOM 从左，看到这里就明白，原来 Vue 是这样动态创建 DOM 的。

在 createElm 过程中，如果 vnode 节点不包含 tag，则它有可能是一个注释或者文本节点，可以直接插入到元素中。在我们的例子中，最内层就是一个文本 vnode，它的 text 值取的就是之前的 this.message 的值 hello vue!

再回到 patch 方法，首次渲染我们调用了 createElm 方法，这里传入的 parentElm 是 oldVnode.elm 的父元素，在我们的例子是 id 为`#app` 的 div 的父元素，也就是 body 实际上，整个过程就是递归创建了一个完整的 dom 树并插入到 body 上。

最后我们根据之前的 createElm 生成了 vnode 插入顺序队列，执行相关的 insert 钩子函数

## 总结

那么至此我们从主线上把模版和数据如何渲染成最终的 DOM 的过程解析完毕了。我们可以直接通过下图更直观地看到从初始化 Vue 到最终渲染的整个过程。

![初始化Vue到最终渲染的整个过程](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)

我们这里只是分析了最简单和最基础的场景，在实际项目中，我们是把页面拆成很多组件的，Vue 另一个核心思想就是组件化。那么下一章我们就来分析 Vue 的组件化过程。
