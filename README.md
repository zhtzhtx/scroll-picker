# scroll-picker

使用Vue编写的滑动选择器，支持懒加载功能。

## 示例

```vue
<template>
  <div id="app">
    <ScrollPicker style="height:300px" :data="dataList"></ScrollPicker>
    <ScrollPickerLazy style="height:300px" :data="longList" :limit="5"></ScrollPickerLazy>
  </div>
</template>

<script>
import ScrollPicker from './components/ScrollPicker'
import ScrollPickerLazy from './components/ScrollPickerLazy'

export default {
  name: 'App',
  components: {
    ScrollPicker,
    ScrollPickerLazy
  },
  data() {
    return {
      dataList: [
        {
          label: "1"
        },
        {
          label: "2"
        },
        {
          label: "3"
        },
        {
          label: "4"
        },
        {
          label: "5"
        },
      ],
      longList: [
        {
          label: "1"
        },
        {
          label: "2"
        },
        {
          label: "3"
        },
        {
          label: "4"
        },
        {
          label: "5"
        },
        {
          label: "6"
        },
        {
          label: "7"
        },
        {
          label: "8"
        },
        {
          label: "9"
        },
        {
          label: "10"
        },
        {
          label: "11"
        },
        {
          label: "12"
        },
        {
          label: "13"
        },
        {
          label: "14"
        }
      ]
    }
  }
}
</script>
```

