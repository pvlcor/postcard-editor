<template>
  <div class="root">
    <div class="header">
      <div class="title">Awesome Postcard Editor</div>
    </div>
    <div class="body">
      <PostcardEditForm />
    </div>
  </div>
</template>

<script>
import PostcardEditForm from './components/PostcardEditForm.vue';

export default {
  components: {
    PostcardEditForm,
  },
};
</script>

<style>
  body {
    font-family: Arial, Helvetica, sans-serif;
    font-size: 14px;
    padding: 0;
    margin: 0;
  }

  .root {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100vh;
  }

  .header {
    vertical-align: middle;
    height: 64px;
    line-height: 64px;
    background-color: cornflowerblue;
    box-shadow: 0 0 8px 0px rgb(0 0 0 / 75%);
  }

  .title {
    color: white;
    font-size: 24px;
    font-weight: bold;
    letter-spacing: 0.5px;
    margin-left: 16px;
  }

  .body {
    background-color: #fefefe;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
