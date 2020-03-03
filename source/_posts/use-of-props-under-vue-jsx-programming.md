---
title: Vue jsx编程下Props的使用
date: 2019-03-03 16:27:34
tags: ["vue"]
category:
  - vue
---

# 开启 JSX 编程

类似 React 的 Props 传参数、方法，实现父子组件通信与数据绑定

主要通过 Props 传递方法，从而节省 `this.$emit()`

缺点就是没法像 React 那样 `this.props.xxx` 直接赋值，Vue 的 props 必须要配置才能获取到参数

[Demo 链接]

```javascrip
# App.vue

<script>
import Child from "./Child";
export default {
  data() {
    return {
      msg: "default"
    };
  },
  methods: {
    handleChange(v) {
      console.log(v.target.value);
      this.msg = v.target.value;
    }
  },
  render() {
    return (
      <div class="parent">
        <p>父组件</p>
        <p>{this.msg}</p>
        <Child msg={this.msg} handleChange={this.handleChange} />
      </div>
    );
  }
};
</script>

<style>
.parent {
  border: 1px #000 solid;
}
</style>

```

```javascript
# Child.vue
<script>
export default {
  props: ["msg", "handleChange"],
  render() {
    return (
      <div class="child">
        <p>子组件</p>
        <p>msg: {this.msg}</p>
        <input value={this.msg} onInput={this.handleChange} />
      </div>
    );
  }
};
</script>
<style>
.child {
  border: 1px #ccc solid;
}
</style>

```

# 组件以 Props 传递

```javascript
# App.vue
<script>
import HelloWorld from "./components/HelloWorld";
import Child from "./Child";
export default {
  render() {
    return (
      <div class="parent">
        <p>父组件</p>
        <Child
          component={HelloWorld}
        />
      </div>
    );
  }
};
</script>

<style>
.parent {
  border: 1px #000 solid;
}
</style>

```

使用`h()`渲染组件

```javascript
# Child.vue

<script>
export default {
  props: ["component"],
  render(h) {
    return (
      <div class="child">
        <p>子组件</p>
        <p>msg: {this.msg}</p>
        <input value={this.msg} onInput={this.handleChange} />

        <p>父组件传过来的组件</p>
        {h(this.component)}
      </div>
    );
  }
};
</script>
<style>
.child {
  border: 1px #ccc solid;
}
</style>

```
