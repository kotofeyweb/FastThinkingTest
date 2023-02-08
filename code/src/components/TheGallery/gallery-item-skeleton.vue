<template>
  <li class="gallery-item-skeleton">
    <div class="gallery-item-skeleton__img-wrap"></div>

    <div class="gallery-item-skeleton__label"></div>
  </li>
</template>

<style scoped lang="scss">
@mixin loadingBgAnimation {
  border-radius: 6px;
  position: relative;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.1);

  &::before {
    content: "";
    width: 100%;
    height: 100%;
    position: absolute;
    inset: 0 auto auto 0;
    background: linear-gradient(
      90deg,
      rgba(0, 0, 0, 0) 0%,
      rgba(0, 0, 0, 0.2) 50%,
      rgba(0, 0, 0, 0) 100%
    );
    animation-duration: 0.6s;
    animation-name: moveBgFromLeftToRight;
    animation-iteration-count: infinite;
    animation-timing-function: ease-in-out;
    transform-origin: center center;
  }

  @keyframes moveBgFromLeftToRight {
    0% {
      left: -100%;
    }
    100% {
      left: 100%;
    }
  }
}

// Gallery item skeleton ======================================================
.gallery-item-skeleton {
  width: 160px;

  &__img-wrap {
    @include loadingBgAnimation;
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

  &__label {
    @include loadingBgAnimation;
    width: 100%;
    height: 15px;
    position: relative;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
}
</style>
