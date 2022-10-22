<template>
  <div id="app">
    <ScrollPicker style="height:300px" :data="dataList" ></ScrollPicker>
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
      longList: []
    }
  },
  mounted() {
    const list = []
    for (let i = 1; i < 10000; ++i) {
      list.push({ label: `${i}` })
    }
    this.longList = list
  }
}
</script>

