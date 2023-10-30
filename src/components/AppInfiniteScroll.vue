<script setup>
import { onBeforeMount, ref } from "vue";
import { Octokit } from "octokit";
import BlockRepoCard from "./BlockRepoCard.vue";

const octokit = new Octokit();
const dataPerPage = ref(30);
const nowPage = ref(1);
const repoData = ref([]);

async function getRepoData() {
  const response = await octokit.request(
    `GET /orgs/vuejs/repos?per_page=${dataPerPage.value}&page=${nowPage.value}`,
    {
      org: "vuejs",
      headers: {
        "X-GitHub-Api-Version": "2022-11-28",
      },
    }
  );

  if (response.status === 200) {
    response.data.forEach((el) => {
      repoData.value.push({
        title: el.name,
        description: el.description,
        url: el.html_url,
      });
    });
  }
  console.log(repoData.value);
}

onBeforeMount(async () => {
  getRepoData().then(() => {
    if (dataPerPage.value === 30) dataPerPage.value = 10;
    if (nowPage.value === 1) {
      nowPage.value = 4;
    } else {
      nowPage.value++;
    }
  });
});
</script>

<template>
  <section>
    <h1>
      repo資料來源：<a
        href="https://github.com/vuejs"
        target="_blank"
        rel="noopener noreferrer"
        >https://github.com/vuejs</a
      >
    </h1>
    <div class="card-wrapper">
      <template v-for="card in repoData" :key="card.title">
        <BlockRepoCard v-bind="card"></BlockRepoCard>
      </template>
    </div>
  </section>
</template>

<style scoped lang="scss">
section {
  width: calc(100% - 32px);
  max-width: 1280px;
  margin: 0 auto;
}

.card-wrapper {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}
</style>
