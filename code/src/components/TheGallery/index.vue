<template>
  <div class="images-list">
    <ul class="images-list__list-itself">
      <!-- Single image -->
      <li
        v-for="image in galleryItems"
        :key="image.url + image.label"
        class="single-image images-list__single-image"
      >
        <div class="single-image__img-wrap">
          <img
            class="single-image__img"
            :src="image.isBrokenUrl ? placeholderImage : image.url"
            :alt="image.label"
            :onerror="() => handleBrokenImgUrl(image)"
          />
        </div>

        <div class="single-image__label">
          {{ image.label }}
        </div>
      </li>
      <!-- / Single image -->
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch } from "vue";
import placeholderImage from "@/assets/no-image.png";

interface IGalleryItem {
  url: string;
  label: string;
  isBrokenUrl: boolean;
}

export default defineComponent({
  setup() {
    let galleryItems = ref<IGalleryItem[]>([]);

    onMounted(loadImages);

    async function loadImages<Promise>() {
      const galleryItemsFetched = await fetch(
        "https://s3.eu-west-2.amazonaws.com/assets-test.fast-thinking.co.uk/recruitment/data.json"
      );

      galleryItems.value = await galleryItemsFetched.json();

      galleryItems.value = galleryItems.value.map(
        (item: IGalleryItem): IGalleryItem => ({
          ...item,
          isBrokenUrl: false,
        })
      );
    }

    function handleBrokenImgUrl(brokenItem: IGalleryItem) {
      let brokenItemIndex = galleryItems.value.findIndex(
        (item) => item.url === brokenItem.url && item.label === item.label
      );

      galleryItems.value[brokenItemIndex] = {
        ...galleryItems.value[brokenItemIndex],
        isBrokenUrl: true,
      };
    }

    let reloadingTryCount = 0;
    const reloadingTryCountLimit = 9;
    let reloadTryTimeoutMultiplier = 2000;

    const tryToLoadImagesAgainDebounced = () => {
      let timer: ReturnType<typeof setTimeout>;

      return (...args: []): void => {
        clearTimeout(timer);
        timer = setTimeout(() => {
          tryToLoadImagesAgain.apply(this, args);
        }, reloadTryTimeoutMultiplier * (reloadingTryCount + 1));
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
      placeholderImage,
      handleBrokenImgUrl,
    };
  },
});
</script>

<style scoped lang="scss">
// Single image ===============================================================
.single-image {
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
    position: relative;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
}

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
