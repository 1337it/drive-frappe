<template>
  <LoadingIndicator
    v-show="loading"
    class="w-10 h-full text-neutral-100 mx-auto" />
  <video
    v-show="!loading"
    class="w-auto max-h-full"
    autoplay
    muted
    preload="none"
    controlslist="nodownload noremoteplayback noplaybackrate disablepictureinpicture"
    controls
    :key="src"
    ref="mediaRef"
    draggable="false"
    @loadedmetadata="handleMediaReady">
    <source :src="src" :type="type" />
  </video>
</template>

<script setup>
/* 
  Add codec evaluation currently assumes its a valid H264/5 (MP4/Webm)
  Look into the feasibility of client side mp4 moov fragmentation pre processing using
  https://github.com/gpac/gpac/wiki/MP4Box
  Server side byte is good enough for now 
*/

import { LoadingIndicator } from "frappe-ui";
import { ref, onBeforeUnmount, watch } from "vue";

const props = defineProps(["previewEntity"]);
const loading = ref(true);
const src = ref(
  `/api/method/drive.api.files.get_file_content?entity_name=${props.previewEntity.name}`
);
const type = ref("video/mp4");
const mediaRef = ref("");

const handleMediaReady = (event) => {
  mediaRef.value = event.target;
  if (mediaRef.value.readyState === 1) {
    loading.value = false;
  }
};

watch(
  () => props.previewEntity,
  (newValue, oldValue) => {
    loading.value = true;
    src.value = `/api/method/drive.api.files.get_file_content?entity_name=${newValue.name}`;
    type.value = newValue.mime_type;
  }
);

onBeforeUnmount(() => {
  loading.value = true;
  src.value = "";
  type.value = "";
});
</script>
