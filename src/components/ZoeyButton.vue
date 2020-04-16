<template>
  <button
    class="zoey-button"
    :disabled="disabled"
    :class="['zoey-button--' + type, disabled ? 'is-disabled' : '']"
    @click="handleButton">
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'ZoeyButton',
  props: {
    type: {
      type: String,
      default: "priamry"
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    // 触发click事件
    handleButton(evt) {
      this.$emit("click", evt);
    }
  },
}
</script>

<style>
/* zoey-button基础样式 */
.zoey-button {
  padding: 7px 15px;
  cursor: pointer;
  outline: none;
  border: none;
}

.zoey-button:active {
  outline: none;
}

.zoey-button:focus {
  outline: none;
}

/* zoey-button禁止状态基础样式 */
.zoey-button.is-disabled {
  cursor: not-allowed;
}

/* type为primary的zoey-button的样式  */
.zoey-button--primary {
  color: #fff;
  font-size: 22px;
  background-color: #fc798f;
}

/* type为primary的zoey-button处于active状态的样式 */
.zoey-button--primary:active {
  background-color: #f05d75;
}

/* type为primary的zoey-button处于禁用状态的样式 */
.zoey-button--primary.is-disabled {
  color: #6b6c6e;
  background-color: #fc798f;
}

/* type为text的zoey-button的样式  */
.zoey-button--text {
  color: #f6f6f6;
  font-size: 20px;
  background: transparent;
}

/* type为text的zoey-button处于active状态的样式 */
.zoey-button--text:active {
  color: #F4A460;
}

/* type为text的zoey-button处于禁用状态的样式 */
.zoey-button--text.is-disabled {
  color: #c0c4cc;
}
</style>