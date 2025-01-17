<template>
  <q-form @submit="authorizeApiKey">
    <div class="row full-width no-wrap items-center q-gutter-sm">
      <q-input
        outlined
        dense
        :type="showApiKey ? 'text' : 'password'"
        v-model="apiKey"
        label="API Key"
        class="full-width"
        @update:model-value="isAuthorized = false"
      />

      <!-- Toggle hide/show Api Key -->
      <q-btn
        flat
        round
        :icon="showApiKey ? 'visibility_off' : 'visibility'"
        dense
        color="grey"
        @click="showApiKey = !showApiKey"
      />

      <!-- Api Key help popyp -->
      <ApiKeyHelpPopyp />

      <q-btn
        :disable="isAuthorized"
        no-caps
        type="submit"
        :color="isAuthorized ? 'green-5' : 'primary'"
        :label="isAuthorized ? 'Authorized' : 'Authorize'"
        :icon="isAuthorized ? 'lock_open' : 'key'"
        style="min-width: 150px"
      />
    </div>
  </q-form>
</template>

<script setup>
/**
 * User shouldn't see the API key in URL query, so in this component there
 * will never by any function update the url query.
 *
 * @emits setApiKey: emit an event pass the api key to parent component
 */
import { ref, onMounted, watch } from "vue";
import { useRouter, useRoute } from "vue-router";

import ApiKeyHelpPopyp from "src/components/ApiKeyHelpPopup.vue";

const apiKey = ref();
const showApiKey = ref(false);

const isAuthorized = ref(false);

const router = useRouter();
const route = useRoute();
const emit = defineEmits(["setApiKey"]);

const initApiKey = () => {
  /**
   * By default, this will always emit a setApiKey event to parent
   * component.
   */
  if (route.query.apiKey) {
    // Get api key from route and store to local storage
    apiKey.value = route.query.apiKey;
    localStoreApiKey(apiKey.value);
  } else {
    // Get api key from local storage
    apiKey.value = localGetApiKey();
  }

  if (apiKey.value) {
    emitSetApiKey();
  }
};

const localStoreApiKey = () => {
  localStorage.setItem("SentientApiKey", apiKey.value);
};

const localGetApiKey = () => {
  // console.log("Local getting api key");
  return localStorage.getItem("SentientApiKey");
};

const authorizeApiKey = () => {
  if (apiKey.value) {
    localStoreApiKey();
    emitSetApiKey();
  }
};

const emitSetApiKey = () => {
  // Pass the api key to parent component
  isAuthorized.value = true;
  emit("setApiKey", apiKey.value);
};

onMounted(() => initApiKey());

watch(
  () => route.query.apiKey,
  () => initApiKey()
);
</script>

<style lang="scss" scoped></style>
