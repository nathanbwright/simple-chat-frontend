<template>
  <li>
    <p>{{ message }}</p>
  </li>
</template>

<script>
export default {
  name: 'ChatMessage',
  props: ["message"]
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
