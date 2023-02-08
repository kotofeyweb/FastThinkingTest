<template>
  <div class="images-list">
    <ul class="images-list__list-itself">
      <gallery-item
        class="single-image images-list__single-image"
        v-for="image in galleryItems"
        :key="image.url + image.label"
        :data="image"
        @img-load-error="handleBrokenImgUrl"
      />
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch } from "vue";

import type { IGalleryItem } from "@/models/gallery.models";

// Components
import GalleryItem from "./gallery-item.vue";

export default defineComponent({
  components: {
    GalleryItem,
  },

  setup() {
    let galleryItems = ref<IGalleryItem[]>([]);

    onMounted(loadImages);

    async function loadImages<Promise>() {
      let response;

      try {
        response = await fetch(
          "https://s3.eu-west-2.amazonaws.com/assets-test.fast-thinking.co.uk/recruitment/data.json"
        );
      } catch (e) {
        console.error(e);
      }

      let _galleryItems = [];

      if (response?.ok) {
        _galleryItems = await response.json();
      } else {
        console.log(`HTTP Response Code: ${response?.status}`);
      }

      galleryItems.value = _galleryItems.map(
        (item: IGalleryItem): IGalleryItem => ({
          ...item,
          isBrokenUrl: false,
        })
      );
    }

    function handleBrokenImgUrl(brokenItem: IGalleryItem) {
      let brokenItemIndex = galleryItems.value.findIndex(
        (item: IGalleryItem) =>
          item.url === brokenItem.url && item.label === item.label
      );

      galleryItems.value[brokenItemIndex] = {
        ...galleryItems.value[brokenItemIndex],
        isBrokenUrl: true,
      };
    }

    let reloadingTryCount = 0;
    const reloadingTryCountLimit = 9;

    const tryToLoadImagesAgainDebounced = () => {
      let timeoutReference: ReturnType<typeof setTimeout>;

      return (): void => {
        clearTimeout(timeoutReference);
        timeoutReference = setTimeout(
          tryToLoadImagesAgain,
          Math.pow(2, reloadingTryCount + 1) * 1000
        );
      };
    };

    function tryToLoadImagesAgain() {
      if (reloadingTryCount >= reloadingTryCountLimit) {
        return;
      }
      reloadingTryCount += 1;
      loadImages();
    }

    watch(galleryItems, tryToLoadImagesAgainDebounced(), {
      deep: true,
    });

    return {
      galleryItems,
      handleBrokenImgUrl,
    };
  },
});
</script>

<style scoped lang="scss">
// Images list ================================================================
.images-list {
  display: flex;
  justify-content: center;

  &__list-itself {
    width: 100%;
    padding: 0;
    margin: 0;
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    list-style: none;
  }

  &__single-image {
    margin: 6px;
  }
}
</style>
