---
title: 【element-ui】el-cascader 级联选择器清空初始化
date: 2022-11-29 17:20:17
tags:
    - element-ui
---

```vue
<template>
  <el-cascader
    v-model="value"
    :options="options"
    :props="{ checkStrictly: true }"
    ref="cascader"
    @change="handleChange">
  </el-cascader>
</template>
<script>
  export default {
    data() {
      return {
        value: "",
        options: [{
          value: 'zhinan',
          label: '指南',
          children: [{
            value: 'shejiyuanze',
            label: '设计原则',
            children: [{
              value: 'yizhi',
              label: '一致'
            }, {
              value: 'fankui',
              label: '反馈'
            }, {
              value: 'xiaolv',
              label: '效率'
            }, {
              value: 'kekong',
              label: '可控'
            }]
          }, {
            value: 'daohang',
            label: '导航',
            children: [{
              value: 'cexiangdaohang',
              label: '侧向导航'
            }, {
              value: 'dingbudaohang',
              label: '顶部导航'
            }]
          }]
        }]
      };
    },
    methods: {
      handleChange() {
        this.value = "";
        this.$refs["cascader"].checkedValue = [];
      }
    }
  };
</script>
```
