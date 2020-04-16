<template>
  <input :value="value" @change="handleInput($event)" @keyup.enter="addComment" @input="handleBUtton($event)"/>
</template>

<script>
export default {
  name: 'ZoeyInput',
  props: {
    // value属性
    value: {
      type: String,
    }
  },
  methods: {
    handleInput(e) {
      // 触发change事件
      this.$emit("input", e.target.value);
    },
    handleBUtton(e) {
      if(e.target.value !== '' && e.target.value !== undefined) {
        this.$parent.isEmpty = true;
      }
      else
        this.$parent.isEmpty = false;
    },
    addComment() {
      this.$parent.addComment();
    }
  }
}
</script>
  
<style>

</style>