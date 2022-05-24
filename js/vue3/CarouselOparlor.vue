<script setup>
import { computed, onMounted, ref, watch } from 'vue';

import { debounce } from '@/utilities/delay.js';

import Button from '@/components/Button.vue';

const props = defineProps(
  {
    cork: {
      type: Boolean,
      default: false
    },
    overlay: {
      type: Boolean,
      default: false
    }
  }
);

const root = ref();
const content = ref();

const activeContentIndex = ref(0);
const contentInlineSize = ref(0);
const corkInlineSize = ref(0);
const preventScrollDebounce = ref(false);

const contentElements = computed(() => content.value?.querySelectorAll(':scope > :not(:last-child)') ?? []);

const contentResizeObserver = new ResizeObserver(entries => contentInlineSize.value = entries[0].contentBoxSize.inlineSize ?? entries[0].contentBoxSize[0].inlineSize);

const nextImage = () => {
  preventScrollDebounce.value = true;

  if(activeContentIndex.value >= contentElements.value.length - 1)
    return void (activeContentIndex.value = 0);

  if(contentElements.value[activeContentIndex.value + 1].offsetLeft > content.value.scrollWidth - content.value.offsetWidth)
    return void (activeContentIndex.value = 0);

  activeContentIndex.value++;
};

const previousImage = () => {
  preventScrollDebounce.value = true;

  if(activeContentIndex.value <= 0)
    return void (activeContentIndex.value = Number([ ...contentElements.value.entries() ].reverse().find(entry => contentElements.value[entry[0]].offsetLeft <= content.value.scrollWidth - content.value.offsetWidth)?.[0] ?? contentElements.value.length - 1));

  if(contentElements.value[activeContentIndex.value - 1].offsetLeft > content.value.scrollWidth - content.value.offsetWidth)
    return void (activeContentIndex.value = Number([ ...contentElements.value.entries() ].reverse().find(entry => entry[0] < activeContentIndex.value - 1 && contentElements.value[entry[0]].offsetLeft < content.value.scrollWidth - content.value.offsetWidth)?.[0] ?? 0));

  activeContentIndex.value--;
};

const scroll = debounce(
  () => {
    if(preventScrollDebounce.value)
      return void (preventScrollDebounce.value = false);

    if(!props.cork && content.value.scrollLeft == content.value.scrollWidth - content.value.offsetWidth)
      return void (activeContentIndex.value = contentElements.value.length - 1);

    for(let index in contentElements.value) {
      const value = contentElements.value[index];

      index = Number(index);

      if(Math.abs(content.value.scrollLeft - value.offsetLeft) <= 0.5 * value.offsetWidth) {
        if(activeContentIndex.value != index)
          activeContentIndex.value = index;

        break;
      }
    }
  },
  50
);

watch(
  activeContentIndex,
  (value, oldValue) => {
    if(value != oldValue)
      content.value?.scrollTo({ behavior: 'smooth', left: contentElements.value[value]?.offsetLeft ?? 0 });
  }
);

watch(
  () => props.cork,
  value => {
    if(!content.value)
      return 0;

    if(value)
      corkInlineSize.value = contentElements.value[contentElements.value.length - 1].offsetWidth < content.value.offsetWidth ? content.value.offsetWidth - contentElements.value[contentElements.value.length - 1].offsetWidth : 0;
    else {
      let element = [ ...contentElements.value ].reverse().find(element => element.offsetLeft <= content.value.scrollWidth - content.value.offsetWidth);

      corkInlineSize.value = content.value.scrollWidth - element.offsetLeft - content.value.offsetWidth;
    }
  }
);

onMounted(
  () => {
    contentResizeObserver.observe(content.value);

    setTimeout(
      () => {
        if(props.cork)
          corkInlineSize.value = contentElements.value[contentElements.value.length - 1].offsetWidth < content.value.offsetWidth ? content.value.offsetWidth - contentElements.value[contentElements.value.length - 1].offsetWidth : 0;
        else {
          let element = [ ...contentElements.value ].reverse().find(element => element.offsetLeft <= content.value.scrollWidth - content.value.offsetWidth);

          corkInlineSize.value = content.value.scrollWidth - element.offsetLeft - content.value.offsetWidth;
        }
      },
      50
    );
  }
);
</script>


<template>
<div
  data-component="Carousel"
  :data-overlay="props.overlay || undefined"
  ref="root"
  :style="{ '--Carousel-inline-size': `${contentInlineSize}px` }"
>
  <div class="controls">
    <Button
      class="controls__button"
      @click="previousImage"
    >
      <span class="controls__button__icon feather feather-chevron-left"></span>
    </Button>

    <Button
      class="controls__button"
      @click="nextImage"
    >
      <span class="controls__button__icon feather feather-chevron-right"></span>
    </Button>
  </div>

  <div
    class="content"
    ref="content"
    @scroll="scroll"
  >
    <slot/>

    <div class="content__cork" :style="{ 'inline-size': `${corkInlineSize}px` }"></div>
  </div>
</div>
</template>

<style lang="scss" scoped>
[data-component="Carousel"] {
  block-size: 100%;
  inline-size: 100%;
  overflow: hidden;

  .controls {
    align-items: center;
    block-size: 100%;
    display: flex;
    inline-size: 100%;
    inset-block-start: 0;
    inset-inline-start: 0;
    justify-content: space-between;
    pointer-events: none;
    position: absolute;
    z-index: 2;

    & > * {
      pointer-events: auto;
    }
  }

  .controls__button {
    --_background-color: hsl(var(--theme-color-brand));
    --_border-color: var(--_background-color);
    --_color: hsl(var(--theme-color-brand-complement));

    --hovered_background-color: var(--_background-color);
    --hovered_border-color: var(--_border-color);
    --hovered_color: var(--_color);

    border: none;
    border-radius: 0;
    inline-size: 7.5em;
    padding: 0;
  }

  .controls__button__icon {
    font-size: 7.5em;
  }

  .content {
    block-size: 100%;
    inline-size: calc(100% - 15em);
    inset-inline-start: 7.5em;
    overflow: auto hidden;
    scrollbar-width: none;
    scroll-snap-type: x mandatory;
    white-space: nowrap;
    z-index: 1;
    -ms-overflow-style: none;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  .content__cork {
    block-size: 0;
  }

  &[data-overlay] {
    overflow-x: visible;

    .controls__button {
      --_background-color: hsl(var(--theme-color-brand));
      --_border-color: var(--_background-color);
      --_color: hsl(var(--theme-color-brand-complement));

      --hovered_background-color: var(--_background-color);
      --hovered_border-color: var(--_border-color);
      --hovered_color: var(--_color);

      inline-size: 3em;
      padding-block: 0.5em;

      &:first-child {
        margin-inline-start: 2em;
      }

      &:last-child {
        margin-inline-end: 2em;
      }
    }

    .controls__button__icon {
      font-size: 2em;
    }
  }
}
</style>

<style lang="scss">
[data-component="Carousel"] {
  .content {
    & > * {
      display: inline-block;
      scroll-snap-align: start;
    }

    & > .shop-category {
      display: inline-grid;
      grid-template-columns: [fullbleed-start] 1fr [fullbleed-end];
      grid-template-rows: [image-start] 1fr [description-start image-end] auto [description-end];
      inline-size: calc(25% - 0.75em);

      &:not(:first-child) {
        margin-inline-start: 1em;
      }

      .shop-category__image {
        aspect-ratio: 3 / 4;
        inline-size: 100%;
      }

      .shop-category__description {
        background-color: hsl(var(--theme-color-foreground));
        font-size: var(--theme-font-size-extra-large);
        font-weight: bold;
        grid-area: description / fullbleed;
        padding: 1em;
        text-align: center;
      }
    }
  }
}
</style>
