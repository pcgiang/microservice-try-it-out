<template>
  <DefaultDocUrls @select-doc-url="(url) => pushUrlToRoute(url)" />
  <TestingDocUrls @select-doc-url="(url) => pushUrlToRoute(url)" />
  <q-form @submit="pushDocUrlToRoute">
    <div class="row full-width no-wrap items-center q-gutter-sm">
      <q-input
        outlined
        dense
        v-model="docUrl"
        label="Document Url"
        class="full-width"
      />
      <q-btn
        no-caps
        color="primary"
        label="Import Document"
        type="submit"
        style="min-width: 150px"
      />
    </div>
  </q-form>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import { useRouter, useRoute } from "vue-router";

import DefaultDocUrls from "src/components/Debug/DefaultDocUrls.vue";
import TestingDocUrls from "src/components/Debug/TestingDocUrls.vue";

const docUrl = ref();

const router = useRouter();
const route = useRoute();

const pushDocUrlToRoute = () => {
  router.push({
    query: {
      docUrl: encodeURI(docUrl.value),
    },
  });
};

const getDocUrlFromRoute = () => {
  if (route.query.docUrl) {
    docUrl.value = decodeURI(route.query.docUrl);
  }
};

const pushUrlToRoute = (url) => {
  // console.log("pushUrlToRoute\n", encodeURI(url));
  router.push({
    query: {
      docUrl: encodeURI(url),
    },
  });
};

onMounted(() => getDocUrlFromRoute());

watch(
  () => route.query.docUrl,
  () => getDocUrlFromRoute()
);
</script>

<style lang="scss" scoped></style>
