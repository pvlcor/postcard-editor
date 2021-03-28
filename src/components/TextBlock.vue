<template>
  <div
    class="root"
    contenteditable
    @blur="onBlur"
    v-html="value"
  ></div>
</template>

<script>
export default {
  model: {
    prop: 'value',
    event: 'update:value',
  },
  props: {
    value: {
      type: String,
      default: '',
    },
  },
  methods: {
    onBlur(event) {
      this.$emit('update:value', this.$el.textContent);
      this.$emit('blur', event);
    },
    focus() {
      this.$el.focus();
    },
  },
};
</script>

<style scoped>
  .root {
    background: transparent;
    min-height: 30px;
    height: auto;
    padding: 8px;
    outline: 0;
    transition: background 150ms ease-in-out;
    color: white;
    text-align: center;
    font-size: 24px;
    border-radius: 4px;
  }

  .root:focus {
    background: rgb(255 255 255 / 47%);
    color: black;
  }
</style>
