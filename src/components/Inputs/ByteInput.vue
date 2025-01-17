<template>
  <div>
    <div class="row no-wrap items-stretch">
      <div
        class="base64_preview_area full-width"
        @click="showEditPopupAlert = true"
      >
        <p class="q-ma-none" v-if="clippedStr">
          {{ clippedStr }}
        </p>
        <p v-else class="text-grey-5">No Data</p>
      </div>
      <div class="q-px-xs column justify-center q-gutter-md">
        <q-btn
          icon="upload"
          flat
          dense
          color="primary"
          @click="showUploader = true"
        >
          <q-tooltip> Upload Binary File </q-tooltip>
        </q-btn>
        <!-- TODO: To enable link uploader feature -->
        <!-- <q-btn icon="link" flat dense color="primary">
          <q-tooltip> Upload File Via Link</q-tooltip>
        </q-btn> -->
        <q-btn
          icon="preview"
          flat
          dense
          color="primary"
          @click="showBase64Viewer = true"
        >
          <q-tooltip> Preview base64 media </q-tooltip>
        </q-btn>

        <!-- if base64 media type is audio -->
        <q-btn
          v-if="mediaType == 'audio'"
          icon="mic"
          flat
          dense
          color="primary"
          @click="showVoiceRecorder = true"
        >
          <q-tooltip> Record Voice </q-tooltip>
        </q-btn>
        <slot name="operations"></slot>
      </div>
    </div>

    <!-- Edit base64 alert -->
    <q-dialog v-model="showEditPopupAlert">
      <q-card class="q-pa-md">
        <p>
          You are trying to edit a long base64 string.
          <b
            >Doing this may slow down your computer or completely freeze your
            browser.</b
          >
        </p>
        <p>
          If you are trying to change the file content, please consider use the
          file uploading function instead of direct modify the base64 string.
        </p>
        <div class="row justify-center q-gutter-md">
          <q-btn
            no-caps
            label="Cancel"
            flat
            @click="showEditPopupAlert = false"
          />
          <q-btn
            flat
            no-caps
            color="white"
            class="bg-primary"
            label="Edit Base64 String"
            @click="
              {
                showEditPopupAlert = false;
                showEditWindow = true;
              }
            "
          />
        </div>
      </q-card>
    </q-dialog>

    <!-- Base64 Editor -->
    <q-dialog v-model="showEditWindow" persistent>
      <q-card class="q-pa-md">
        <ByteEditor
          :base64str="base64str"
          @cancel="showEditWindow = false"
          @update="
            (newBase64Str) => {
              showEditWindow = false;
              emitUpdate(newBase64Str);
            }
          "
        />
      </q-card>
    </q-dialog>

    <!-- Binary Uploader -->
    <q-dialog v-model="showUploader" persistent>
      <q-card class="q-pa-md full-width">
        <BinaryUploader
          @cancel="showUploader = false"
          @update="
            (newBase64Str) => {
              showUploader = false;
              emitUpdate(newBase64Str);
            }
          "
        />
      </q-card>
    </q-dialog>

    <!-- Base64 Viewer -->
    <q-dialog v-model="showBase64Viewer">
      <q-card class="q-pa-xs full-width text-center">
        <h6 class="q-ma-sm">Base64 media preview</h6>
        <div
          v-if="base64str"
          class="q-mx-auto row justify-center q-pa-sm q-mb-md"
          style="
            width: clamp(280px, 100%, 400px);
            background-color: rgba(155, 155, 155, 0.25);
          "
        >
          <Base64Viewer :base64str="guessedBase64 || base64str" />
        </div>
        <q-btn label="Close" flat @click="showBase64Viewer = false" />
      </q-card>
    </q-dialog>

    <!-- Audio Recorder -->
    <q-dialog v-model="showVoiceRecorder" persistent>
      <q-card class="q-pa-md full-width">
        <RtcAudioRecorder
          @cancel="showVoiceRecorder = false"
          @base64update="
            (newBase64Str) => {
              showVoiceRecorder = false;
              emitUpdate(newBase64Str);
            }
          "
        />
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
/**
 * @author                                                        zq
 * @lastUpdate                                           2022-May-29
 * -----------------------------------------------------------------
 * A component that allows user to directly edit a long bse64 string
 * optomised for browser to display long strings.      User can also
 * choose to upload a file as base64 or preview the file.
 *
 * User will interactive with popup windows for most of the time.
 * -----------------------------------------------------------------
 * @dependency ByteEditor.vue  - edit large base64 string
 * @dependency BinaryUpoloader.vue - upload a file as base64 str
 * @dependency Base64Viewer.vue - preview of a base64
 *             (only when media header is provided)
 * @dependency guessBase64UriByName.js - help to guess media type of
 *             the base64 string (if there is no media header)
 * -----------------------------------------------------------------
 * @slot operations - insert a small operation button at right side
 *       use an icon-only button.Refer to the 2 button style in this
 *       component right on top of the <slot> tag
 * -----------------------------------------------------------------
 * @props base64str - The raw base64 string,  will be clipped before
 *        display
 * @props trim (default:200) - determine the length of base64 string
 *        to trim, throught this way, we can save viewing space also
 *        not to slow down the browser by rendering long text.
 * @props label - the possible key value of this field, help to guess
 *        the type of base64 media
 * -----------------------------------------------------------------
 * @emit update - emit the updated base64 string to parent component
 *
 */
/**
 * TODO: To enable upload via link feature
 */
import { ref, onMounted, watch, computed } from "vue";

import {
  guessBase64UriByName,
  guessBase64MediaType,
} from "src/services/guessedObjProcessor";

import ByteEditor from "./ByteEditor.vue";
import BinaryUploader from "./BinaryUploader.vue";
import RtcAudioRecorder from "./RtcAudioRecorder.vue";
import Base64Viewer from "src/components/Viewers/Base64Viewer.vue";

const props = defineProps({
  base64str: {},
  trim: { default: 200 },
  // This will help to identify the type of base64 string
  label: {},
});

const emit = defineEmits(["update"]);

const showEditPopupAlert = ref();
const showEditWindow = ref();
const showUploader = ref();
const showBase64Viewer = ref();
const showVoiceRecorder = ref();

const clippedStr = ref();

const guessedBase64 = computed(() => {
  return guessBase64UriByName(props.label, props.base64str);
});

const mediaType = computed(() => {
  return guessBase64MediaType(props.label, props.base64str);
});

const init = () => {
  const invalidVals = [null, undefined];
  if (!invalidVals.includes(props.base64str)) {
    clippedStr.value = clipStr(props.base64str, props.trim);
  }
};

const emitUpdate = (newBase64Str) => {
  // console.log("saveBase64Edit\n", base64EditArea.value.innerText);
  emit("update", newBase64Str);
};

const clipStr = (rawStr, trim) => {
  if (rawStr.length > trim) {
    return `${rawStr.slice(0, trim)}...(${
      rawStr.length - trim
    } characters been clipped)`;
  } else {
    return `${rawStr}`;
  }
};

watch(
  () => props.base64str,
  () => init()
);

onMounted(() => init());
</script>

<style lang="scss" scoped>
.base64_preview_area {
  border: 1px solid rgba(128, 128, 128, 0.3);
  padding: 0.7rem;
  border-radius: 4px;

  cursor: pointer;
}
.base64_preview_area:hover {
  border-color: $primary;
  border-width: 2px;
  box-shadow: 0px;
}
.base64_preview_area p {
  white-space: pre-wrap;
  line-break: anywhere;
  font-size: 0.8rem;
  border: 1px solid rgba(0, 0, 0, 0);
}
.base64_preview_area:hover p {
  border-width: 0px;
}
</style>
