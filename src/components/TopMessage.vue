<template>
    <div class="message">
      <b>{{ message }}</b>
    </div>
</template>

<script>
export default {
  props: {
    message: {
      type: [String],
      default: null,
    }
  },
}
</script>