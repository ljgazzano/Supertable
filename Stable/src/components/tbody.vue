<template>
  <tbody v-html="html"></tbody>
</template>

<script>
export default {
  props: {
    html: String,
  },
};
</script>

<style>
</style>