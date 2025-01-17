<template>
  <div>
    <q-tabs
      dense
      no-caps
      v-model="tab"
      align="justify"
      narrow-indicator
      indicator-color="transparent"
      active-class="text-white bg-beige"
      class="bg-grey-2"
    >
      <q-tab
        v-for="tabKey in Object.keys(tabs)"
        :key="tabKey"
        :name="tabKey"
        :label="tabs[tabKey].label"
      ></q-tab>
    </q-tabs>
    <div style="border: 1px solid #efefef">
      <q-tab-panels v-model="tab">
        <q-tab-panel name="getUploadPolicy">
          <!-- 
            is-sentient-large-file-ms will enable a file
            uploader,it allow user fill in the file then
            read the file size.
           -->
          <RequestsComp
            :show-endpoint-and-method="true"
            :is-sentient-large-file-ms="true"
          />
        </q-tab-panel>

        <q-tab-panel name="uploadFile">
          <!-- Preview Response -->
          <div v-if="apiResponse">
            <q-expansion-item
              dense
              dense-toggle
              class="q-mb-md"
              :class="
                isGetUploadPolicyResSuccess ? 'bg-green-1' : 'bg-yellow-1'
              "
              label="Resposne From Last Step"
              style="border: 1px solid rgba(128, 128, 128, 0.25)"
            >
              <div class="q-ma-sm">
                <RawRes :response="apiResponse" />
              </div>
            </q-expansion-item>

            <small
              class="q-px-sm"
              :class="
                isGetUploadPolicyResSuccess ? 'text-green-6' : 'text-red-6'
              "
              >{{ getUploadPolicyResMsg }}</small
            >
          </div>

          <div>
            <RequestsComp
              :show-endpoint-and-method="true"
              :request-body-example-obj="getUploadPolicyRes"
            />
          </div>
        </q-tab-panel>

        <!-- Status Panel -->
        <q-tab-panel name="status">
          <div v-if="apiResponse">
            <q-expansion-item
              dense
              dense-toggle
              class="q-mb-md"
              :class="
                isGetUploadPolicyResSuccess ? 'bg-green-1' : 'bg-yellow-1'
              "
              label="Resposne From Last Step"
              style="border: 1px solid rgba(128, 128, 128, 0.25)"
            >
              <div class="q-ma-sm">
                <RawRes :response="apiResponse" />
              </div>
            </q-expansion-item>

            <small
              class="q-px-sm"
              :class="
                isGetUploadPolicyResSuccess ? 'text-green-6' : 'text-red-6'
              "
              >{{ uploadFileResMsg }}</small
            >
          </div>
          <div>
            <RequestsComp :show-endpoint-and-method="true" />
          </div>
        </q-tab-panel>

        <q-tab-panel name="response">
          <div class="row justify-between q-mb-sm">
            <div class="row items-center">
              <b>Pretty Response</b>
              <q-toggle
                size="2rem"
                color="green-7"
                class="state-toggle"
                v-model="isRawResponse"
                @click="handleInputMthdSwitch()"
              />
              <b>Raw Response</b>
            </div>

            <!-- Expand all nested pretty response -->
            <div v-if="!isRawResponse">
              <q-checkbox
                size="2rem"
                v-model="prettyResExpandAll"
                label="Expand All"
              />
            </div>
          </div>
          <RawRes v-if="isRawResponse" :response="apiResponse" />
          <PrettyRes v-else :response="apiResponse" />
        </q-tab-panel>
      </q-tab-panels>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref, watch } from "vue";
import { useQuasar } from "quasar";

import { apiResponse } from "src/services/apiCallService";

import {
  largeFile,
  getLargeFilePathByTag,
} from "src/services/largeFileService";

import { prettyResExpandAll } from "src/services/appService";

import { requestBody, setReqBdyExample } from "src/services/apiService";

import RawRes from "src/components/ReqRes/RawRes.vue";
import PrettyRes from "src/components/ReqRes/PrettyRes.vue";
import RequestsComp from "src/components/ReqRes/RequestsComp.vue";

const $q = useQuasar();
const emit = defineEmits(["setApiPath"]);

const tabs = {
  getUploadPolicy: { label: "Get Upload Policy" },
  uploadFile: { label: "Upload File" },
  status: { label: "Status" },
  response: { label: "Response" },
};

const isRawResponse = ref(false);

const tab = ref("getUploadPolicy");

const getUploadPolicyRes = ref();

const uploadFileRes = ref();

const isGetUploadPolicyResSuccess = computed(() => {
  if (getUploadPolicyRes.value) {
    return true;
  } else {
    return false;
  }
});

const getUploadPolicyResMsg = computed(() => {
  // console.log(apiResponse.value);
  const apiFailure =
    apiResponse.value?.["data"]?.["status"].toLowerCase() == "failure";
  if (getUploadPolicyRes.value) {
    return "Response from previous API call be applied to current request body.";
  } else if (apiFailure) {
    return "Error from previous api call. Please check your input.";
  } else {
    return "";
  }
});

const uploadFileResMsg = computed(() => {
  console.log(apiResponse.value);
  const apiFailure = apiResponse.value == "Error: Network Error";
  if (getUploadPolicyRes.value) {
    return "Response from previous API call be applied to current request body.";
  } else if (apiFailure) {
    return "Network error. This is may caused by issues in the endpoint URL.";
  } else {
    return "";
  }
});

const handleUserSelectTab = (tabName) => {
  let pathToSet;
  pathToSet = getLargeFilePathByTag(tabName);

  if (pathToSet) {
    emit("setApiPath", pathToSet);
  }
};

const setGetUploadPolicyRes = () => {
  const apiResFields = apiResponse.value?.["data"]?.["results"]?.["fields"];
  if (apiResFields) {
    getUploadPolicyRes.value = apiResFields;

    // Set stored binary file as the file value
    getUploadPolicyRes.value["file"] = largeFile.value;

    // Store JID, file name, processed time, configruation locally

    $q.notify(getUploadPolicyResMsg.value);
  } else {
    // It could also be the response data structure have been changed
    $q.notify({
      message: getUploadPolicyResMsg.value,
      color: "orange-7",
    });
  }
};

const setUploadFileRes = () => {
  const apiResFields = apiResponse.value?.["data"]?.["results"]?.["fields"];
  if (apiResFields) {
    uploadFileRes.value = apiResFields;

    // Set stored binary file as the file value

    $q.notify(uploadFileResMsg.value);
  } else {
    // It could also be the response data structure have been changed
    $q.notify({
      message: uploadFileResMsg.value,
      color: "orange-7",
    });
  }
};

onMounted(() => {
  console.log("LargeFile.vue Mounted");
  // Init based on first query string
});

watch(tab, () => {
  handleUserSelectTab(tab.value);
});

watch(apiResponse, () => {
  // console.log("Watching apiResponse Change", apiResponse);
  if (apiResponse.value) {
    const tabsArr = Object.keys(tabs);
    const currentTabIndex = tabsArr.findIndex((elem) => {
      return elem == tab.value;
    });
    tab.value = tabsArr[currentTabIndex + 1];

    // If move from 1st api call to 2nd api call
    if (tab.value == tabsArr[1]) setGetUploadPolicyRes();
    // If user moved from 2nd api to 3rd api
    if (tab.value == tabsArr[1]) setUploadFileRes();
  }
});
</script>

<style lang="scss" scoped></style>
