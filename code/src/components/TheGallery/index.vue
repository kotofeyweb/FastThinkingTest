<template>
  <div class="images-list">
    <ul v-if="!isDataLoaded" class="images-list__list-itself">
      <gallery-item-skeleton
        class="single-image images-list__single-image"
        v-for="(_, index) in Array(16)"
      />
    </ul>
    <ul
      v-else-if="isDataLoaded && galleryItemsList.length"
      class="images-list__list-itself"
    >
      <gallery-item
        class="single-image images-list__single-image"
        v-for="(image, index) in galleryItemsList"
        :key="image.id"
        :data="image"
        @img-load-error="handleBrokenImgUrl"
      />
    </ul>

    <div
      v-else-if="isDataLoaded && !galleryItemsList.length"
      class="images-list__no-images-message"
    >
      There is no images to show
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch } from "vue";
import { v4 as uuidv4 } from "uuid";

import type { IGalleryItem } from "@/models/gallery.models";

// Components
import GalleryItem from "./gallery-item.vue";
import GalleryItemSkeleton from "./gallery-item-skeleton.vue";

export default defineComponent({
  components: {
    GalleryItem,
    GalleryItemSkeleton,
  },

  setup() {
    let galleryItemsList = ref<IGalleryItem[]>([]);

    onMounted(loadImages);

    let isDataLoaded = ref<boolean>(false);

    async function loadImages<Promise>(isOnlyUpdateBroken: boolean = false) {
      let response;

      try {
        response = await fetch(
          "https://s3.eu-west-2.amazonaws.com/assets-test.fast-thinking.co.uk/recruitment/data.json"
        );
        isDataLoaded.value = true;
      } catch (e) {
        console.error(e);
      }

      let _galleryItemsList = [];

      if (response?.ok) {
        _galleryItemsList = await response.json();
      } else {
        console.log(`HTTP Response Code: ${response?.status}`);
      }

      if (isOnlyUpdateBroken) {
        const indexesOfBrokenImages: Array<number> =
          galleryItemsList.value.reduce<Array<number>>(
            (acc, item, index) => (!item.isBrokenUrl ? [...acc, index] : acc),
            []
          );

        indexesOfBrokenImages.forEach((i) => {
          galleryItemsList.value[i] = {
            ...galleryItemsList.value[i],
            isBrokenUrl: false,
          };
        });
      } else {
        galleryItemsList.value = _galleryItemsList.map(
          (item: IGalleryItem): IGalleryItem => ({
            ...item,
            id: uuidv4(),
            isBrokenUrl: false,
          })
        );
      }
    }

    function handleBrokenImgUrl(id: string) {
      const brokenUrlItemIndex: number = galleryItemsList.value.findIndex(
        (item) => item.id === id
      );

      galleryItemsList.value[brokenUrlItemIndex] = {
        ...galleryItemsList.value[brokenUrlItemIndex],
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
      loadImages(true);
    }

    watch(galleryItemsList, tryToLoadImagesAgainDebounced(), {
      deep: true,
    });

    return {
      galleryItemsList,
      isDataLoaded,
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

  &__no-images-message {
    color: gray;
  }
}
</style>
