<template>
  <h5 style="margin: 0; font-size: 20px; font-weight: 700">Try It Out</h5>
  <p style="max-width: 800px">
    You can test the input / output of Sentient.io API with the tool we provided
    below. Simply input your subscribed API key, and you’ll be able to test the
    input with the fields and check the output below.
  </p>
  <BeforeYouStart />

  <div class="column q-gutter-lg q-my-md">
    <DocUrlField />
    <!-- TODO: To build a authorize component, which should be able to take multiple types of authentication -->
    <!-- {{ securitySchemes }} -->
    <ApiKeyField @setApiKey="(apiKey) => setApiKey(apiKey)" />
  </div>

  <div v-if="doc && sentientLargeFileMsDetails">
    <div v-if="isSentientLargeFileMs">
      <LargeFileSwitcher
        :enabled="tryAsLargeFile"
        @toggle="
          (mode) => {
            tryAsLargeFile = mode;
          }
        "
      />
    </div>

    <div v-if="tryAsLargeFile" class="q-mt-sm">
      <LargeFile @set-api-path="(path) => setApiPath(path)" />
    </div>

    <div v-else>
      <div class="column q-col-gutter-md">
        <div class="column q-gutter-md q-my-xs q-px-lg">
          <ApiPathSelector
            :api-paths="apiPaths"
            :selected-path="apiPath"
            :server-str="serverStr"
            :server-str-description="serverStrDescription"
            @selectPath="(path) => setApiPath(path)"
          />
          <ListSelector
            label="Method"
            :options="methods"
            :default-value="method"
            @select="(val) => useSetMethod(val)"
          />
          <ListSelector
            label="Content Type"
            :options="contentTypes"
            :default-value="contentType"
            @select="(contType) => useSetContentType(contType)"
          />
        </div>

        <!-- !!Critical Component!! -->
        <RequestResponse v-if="doc" />
      </div>
    </div>
  </div>

  <InlineError :error-message="docErr" v-if="docErr" />
  <!-- TODO Use swagger style of warning box -->
  <InlineError :error-message="yamlErr" v-if="yamlErr" />

  <!-- <pre class="bg-grey-2">{{ api }}</pre> -->
  <VersionNum />
</template>

<script setup>
import { onMounted, watch, ref, computed } from "vue";
import { useRoute } from "vue-router";
import { apiKey, setApiKey } from "src/services/apiKeyService";

// All key functions to extract data from api docs
import {
  loadDoc,
  doc,
  docErr,
  yamlErr,
  getApiPaths,
  getServerStr,
  securitySchemes,
  getApiObjsByPath,
  getMethodListByPath,
  isSentientLargeFileMs,
} from "src/services/docService";

import { sentientLargeFileMsDetails } from "src/services/largeFileService";

import {
  // apis,
  api,
  methods,
  method,
  setApis,
  initApis,
  setMethod,
  requestBody,
  contentType,
  contentTypes,
  setContentType,
} from "src/services/apiService";

import { setEndpoint } from "src/services/tryItOutService";

import DocUrlField from "src/components/DocUrlField.vue";
import BeforeYouStart from "src/components/BeforeYouStart.vue";
import ApiKeyField from "src/components/ApiKeyFiled.vue";
import ApiPathSelector from "src/components/ApiPathSelector.vue";
import InlineError from "src/components/InlineError.vue";
import RequestResponse from "src/components/RequestResponse.vue";
import VersionNum from "src/components/UI/VersionNum.vue";

import ListSelector from "src/modules/ListSelector/ListSelector.vue";

// Components for Large File Microservices
import LargeFile from "src/components/Plus/LargeFile.vue";
import LargeFileSwitcher from "src/components/Plus/LargeFileSwitcher.vue";

const route = useRoute();
const serverObj = ref();

// The pre-fix server string
const serverStr = computed(() => {
  return serverObj.value?.["url"] ?? "";
});

const serverStrDescription = computed(() => {
  return serverObj.value?.["description"] ?? "";
});

// A list of paths under the ['paths'] tag
const apiPaths = ref();
// The path for current selected API
const apiPath = ref();

const tryAsLargeFile = ref(false);

const useLoadDoc = async () => {
  init();
  const docUrl = decodeURI(route.query.docUrl);
  await loadDoc(docUrl);
  if (doc.value) {
    extractDocDetails();
  }
};

const setApiPath = (path) => {
  apiPath.value = path;
};

const extractDocDetails = () => {
  /**
   * !!! Critical Function !!!
   * After doc successfully loaded, extract detail from doc
   */
  apiPaths.value = getApiPaths();
  serverObj.value = getServerStr();
  initApiPath();
};

const serverStrOverride = () => {
  // console.log("serverStrOverride\n", api.value);
  if (api.value?.["servers"]) {
    const serverObjs = api.value?.["servers"];
    serverObj.value = serverObjs[0];
    /**
     * !IMPORTANT!
     * Based on OAS3.0, a server can take more than one element.
     * Current version of try it out will only take the first server
     * element,   and user selecting function is not enabled till we
     * have such microservices (multiple server microservice)
     * 2022 15 Jun by ZQ
     */
    if (serverObjs.length > 1) {
      console.warn(
        "Multiple server element detected. Current version of try it out is not able to handle this."
      );
    }
  } else {
    serverObj.value = getServerStr();
  }
};

const initApiPath = () => {
  /** Init current API path by take the first element in the list */
  if (apiPaths.value[0]) {
    apiPath.value = apiPaths.value[0];
  }
};

const useSetApis = (path) => {
  /** Set api object that belongs to current path */
  // console.log("useSetApis\n", path);

  const apiObjs = getApiObjsByPath(path);
  setApis(apiObjs);
};

const useSetMethod = (meth) => {
  setMethod(meth);
};

const useSetContentType = (contType) => {
  setContentType(contType);
};

const init = () => {
  serverObj.value = null;
  apiPaths.value = null;
  apiPath.value = null;
  initApis();
};

onMounted(() => {
  if (route.query.docUrl) {
    useLoadDoc();
  }
});

watch(
  () => route.query.docUrl,
  () => useLoadDoc()
);

watch(apiPath, () => {
  // console.log("Watching apiPath change\n", apiPath.value);

  const path = apiPath.value;
  if (path) {
    useSetApis(path);
    serverStrOverride();
    setEndpoint(serverObj.value.url, path);
  } else {
    initApis();
  }
});

watch(isSentientLargeFileMs, () => {
  // Default using large file mode if the Ms detected as large file
  tryAsLargeFile.value = isSentientLargeFileMs.value;
});
</script>

<style lang="scss" scoped></style>
