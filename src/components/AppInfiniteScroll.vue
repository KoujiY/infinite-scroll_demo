<script setup>
import { onBeforeMount, onMounted, ref } from "vue";
import { Octokit } from "@octokit/core";
import BlockRepoCard from "./BlockRepoCard.vue";

const octokit = new Octokit();
const dataPerPage = ref(30);
const nowPage = ref(1);
const repoData = ref([]);
const allRepoLoaded = ref(false);
const toastBar = ref(null);
const nowScrollY = ref(0);

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
    // 當接收到的repo數量為小於每次應取得的數量時切換flag
    if (response.data.length < dataPerPage.value) {
      allRepoLoaded.value = true;
      if (response.data.length === 0)
        toastBar.value.classList.add("toast-bar--slide-down");
    }

    // 將接收到的資料push至陣列中
    response.data.forEach((el) => {
      repoData.value.push({
        title: el.name,
        description: el.description,
        url: el.html_url,
      });
    });
  }

  // 設定下一次要增加的資料筆數、當前頁數
  if (!allRepoLoaded.value) {
    if (dataPerPage.value === 30) dataPerPage.value = 10;
    if (nowPage.value === 1) {
      nowPage.value = 4;
    } else {
      nowPage.value++;
    }
  }
}

onBeforeMount(async () => {
  getRepoData();
});

onMounted(() => {
  window.addEventListener("scroll", () => {
    nowScrollY.value = window.scrollY;
    if (window.scrollY + window.innerHeight >= document.body.offsetHeight) {
      if (allRepoLoaded.value) {
        toastBar.value.classList.add("toast-bar--slide-down");
      } else {
        getRepoData();
      }
    }
  });

  window.addEventListener("animationend", () => {
    toastBar.value.classList.remove("toast-bar--slide-down");
  });
});
</script>

<template>
  <section>
    <div class="toast-bar" ref="toastBar" :style="`--scrollY: ${nowScrollY}px`">
      所有repo均已加載完畢
    </div>
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
  padding: 10px 0;
  position: relative;
}

.card-wrapper {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.toast-bar {
  width: fit-content;
  height: fit-content;
  padding: 12px 20px;
  border-radius: 5px;
  border-color: #c3e6cb;
  background-color: #d4edda;
  color: #155724;
  font-size: 16px;
  line-height: 16px;
  position: absolute;
  top: calc(var(--scrollY, 0px) - 100vh);
  left: 50%;
  transform: translateX(-50%);
}

.toast-bar--slide-down {
  animation-name: slide-down;
  animation-duration: 0.8s;
  animation-timing-function: linear;
  animation-direction: alternate;
  animation-fill-mode: both;
  animation-iteration-count: 2;
}

@keyframes slide-down {
  0% {
    top: calc(var(--scrollY, 0px) - 40px);
  }

  37.5% {
    top: calc(var(--scrollY, 0px) + 40px);
  }

  100% {
    top: calc(var(--scrollY, 0px) + 40px);
  }
}
</style>
