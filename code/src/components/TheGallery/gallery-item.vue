<template>
  <li class="gallery-item">
    <div class="gallery-item__img-wrap">
      <img
        class="gallery-item__img"
        :src="props.data.isBrokenUrl ? placeholderImage : props.data.url"
        :alt="props.data.label"
        :onerror="handleBrokenImgUrl"
      />
    </div>

    <div class="gallery-item__label">
      {{ props.data.label }}
    </div>
  </li>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import type { PropType } from "vue";
import type { IGalleryItem } from "@/models/gallery.models";

// Images
import placeholderImage from "@/assets/no-image.png";

export default defineComponent({
  props: {
    data: { type: Object as PropType<IGalleryItem> },
  },

  emits: ["img-load-error"],

  setup(props, { emit }) {
    function handleBrokenImgUrl() {
      emit("img-load-error", props.data?.id);
    }

    return {
      props,
      placeholderImage,
      handleBrokenImgUrl,
    };
  },
});
</script>

<style scoped lang="scss">
// Gallery item ===============================================================
.gallery-item {
  width: 160px;

  &__img-wrap {
    width: 100%;
    height: 140px;
    margin-bottom: 6px;
    border-radius: 6px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    overflow: hidden;
  }

  &__img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  &__label {
    width: 100%;
    height: 15px;
    position: relative;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
}
</style>
