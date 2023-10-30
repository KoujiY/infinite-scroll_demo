<script setup>
const props = defineProps({ title: String, description: String, url: String });
</script>

<template>
  <div class="card">
    <p>
      title：<br /><span>{{ props.title }}</span>
    </p>
    <p>
      description：<br /><span>{{ props.description }}</span>
    </p>
    <p>
      url：<br /><a
        :href="props.url"
        target="_blank"
        rel="noopener noreferrer"
        >{{ props.url }}</a
      >
    </p>
  </div>
</template>

<style scoped lang="scss">
.card {
  width: fit-content;
  max-width: 320px;
  height: fit-content;
  min-height: 20vh;
  padding: 12px;
  border: 1px solid black;
  border-radius: 5px;
  box-sizing: border-box;
  overflow-wrap: break-word;

  p {
    width: 100%;
    font-size: 24px;

    span,
    a {
      font-size: 20px;
    }
  }
}
</style>
