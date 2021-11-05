---
title: 【Vue】全局引入组件，挂载在原型链上
date: 2021-08-04 23:44:01
tags:
    - Vue
    - 全局弹窗
cover_picture: /images/vue.png
---
# 原理
使用```vue.extend()```和```$mount()```
# 弹窗组件
```vue
<template>
  <!-- 遮罩层 也是整个组件的容器-->
  <div class="pop-container" v-if="isShow">
    <div class="message-container">
      <!-- 两个icon放在一个容器中，但是只显示一个 -->
      <div class="icon-container">
        <div class="icon-container-success" v-if="type === 'success'">
          <!-- 引用了iview的Icon组件 -->
          <Icon class="icon-checkmark" type="md-checkmark" size="30" color="#D8DCE9" />
        </div>
        <div class="icon-container-error" v-if="type === 'error'">
          <Icon class="icon-close" type="md-close" size="30" color="#D8DCE9" />
        </div>
      </div>
      <span class="message-text">{{ text }}</span>
    </div>
  </div>
</template>


<script>
export default {
  name: 'message',
  props: {
    type: {               // type控制是成功还是失败
      type: String,
      default: 'success',
    },
    text: {              // 弹窗的文字信息
      type: String,
      default: '',
    },
    isShow: {             // 整个遮罩和弹窗是否显示
      type: Boolean,
      default: true,
    },
  },
};
</script>

<style scoped lang="less">
.pop-container {
  display: flex;
  justify-content: center;
  align-items: center;

  z-index: 99999;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(24, 30, 53, 0.7);
  backdrop-filter: blur(10px);
}
.message-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-width: 384px;
  padding: 0 30px;
  height: 170px;
  background: #303e62;
  box-shadow: 0px 0px 30px rgba(0, 0, 0, 0.1);
  border-radius: 12px;
}
.icon-container {
  position: relative;
  height: 40px;
  width: 40px;
  border-radius: 50%;
  &-error {
    background-color: #fe1b1b;
    height: 40px;
    width: 40px;
    border-radius: 50%;
    .icon-close {
      position: absolute;
      right: 5px;
      bottom: 5px;
      font-weight: 900;
    }
  }
  &-success {
    background-color: #4ad991;
    height: 40px;
    width: 40px;
    border-radius: 50%;
    .icon-checkmark {
      position: absolute;
      right: 4px;
      bottom: 5px;
      font-weight: 900;
    }
  }
}
.message-text {
  margin-left: 12px;
  font-weight: 500;
  font-size: 18px;
  line-height: 27px;
}
</style>
```

# 挂载在原型上
```javascript
    import vue from 'vue';
    // 引入上面写好的组件
    import Message from './message';
    // 全局警告弹窗 第一个参数为提示的文本信息，第二个参数可选，为弹窗持续时间
    
    // 可以提前看一下知识点，Vue.extend + vm.$mount，可以让我们在vue中用js一样的方法去直接调用一个组件
    // https://www.cnblogs.com/hentai-miao/p/10271652.html
    
    // 将vue组件利用Vue.extend方法变成一个组件构造器，相当于一个类
    const MsgClass = vue.extend(Message);
    
    const MsgMain = {
      show(text, type, duration) {
        // 实例化这个组件
        const instance = new MsgClass();
        // 将组件实例挂在到一个元素上面，如果不传参数就是挂载到body，或者也可以传入其他已经存在的元素的选择器
        instance.$mount(document.createElement('div'));
        // 通过组件实例的$el属性，可以访问到这个组件元素，然后把它拼接到body上。
        document.body.appendChild(instance.$el);
        // 给这个实例传入参数
        instance.type = type;
        instance.text = text;
        instance.isShow = true;
        // 设置一个延迟，过了时间弹窗消失
        setTimeout(() => {
          instance.isShow = false;
        }, duration);
      },
      // 成功时调用这个方法
      success(text, duration = 2000) {
        this.show(text, 'success', duration);
      },
      // 失败时调用这个方法
      error(text, duration = 2000) {
        this.show(text, 'error', duration);
      },
    };
    // 全局注册
    function Msg() {
      vue.prototype.$msg = MsgMain;
      // 最终调用就是this.$msg.success() 或者this.$msg.error()
    }
    
    export default Msg;
```

# 全局注册
```javascript
// main.js
import Msg from '../src/components/message';
Vue.use(Msg);
```

# 使用
```javascript
this.$msg.success('设置成功！');   // 第二个参数可以传入弹窗持续时间，默认是2000
this.$msg.success('设置成功！'，3000);
this.$msg.error('设置失败！')
```
