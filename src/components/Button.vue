<template>
  <button
    class="button"
    :disabled="disabled"
    @click="!disabled && $emit('click', $event)"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  props: {
    disabled: {
      type: Boolean,
      default: false,
    },
  },
};
</script>

<style scoped>
  .button {
    width: 150px;
    height: 30px;
    background-color: cornflowerblue;
    text-transform: uppercase;
    outline: none;
    appearance: none;
    border: 0;
    color: white;
    border-radius: 4px;
    cursor: pointer;
    user-select: none;
  }

  .button:hover {
    background-color: #5985d5;
  }

  .button:active {
    background-color: #486cae;
  }

  .button:disabled {
    background-color: lightgray;
    pointer-events: none;
    cursor: default;
  }
</style>
