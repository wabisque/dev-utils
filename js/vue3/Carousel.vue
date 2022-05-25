<script setup>
import { computed, onMounted, ref, watch } from 'vue';

import { debounce, throttle } from '../vanilla/timing.js';

import Button from './Button.vue';

const emits = defineEmits([ 'rotate' ]);
const props = defineProps(
  {
    autoRotate: {
      type: Boolean,
      default: false
    },
    autoRotateTiming: {
      type: Number,
      default: 8000
    },
    columns: {
      type: Boolean,
      default: false
    },
    cork: {
      type: Boolean,
      default: false
    },
    preview: {
      type: Boolean,
      default: false
    }
  }
);

const preview = ref();
const root = ref();
const scroller = ref();

const activeItemIndex = ref(0);
const autoRotating = ref(false);
const nextButtonFocused = ref(false);
const nextButtonHovered = ref(false);
const preventIndexDetection = ref(false);
const previewCorkSize = ref(0);
const previewHovered = ref(false);
const previousButtonFocused = ref(false);
const previousButtonHovered = ref(false);
const scrollerBlockSize = ref(0);
const scrollerCorkSize = ref(0);
const scrollerHovered = ref(false);
const scrollerMousePositionX = ref(0);

const items = computed(() => scroller.value?.querySelectorAll(':scope > :not(:last-child)') ?? []);
const thumbnails = computed(() => preview.value?.querySelectorAll(':scope > :not(:last-child)'));

const previewResizeObserver = new ResizeObserver(
  entries => {
    const lastThumbnail = thumbnails.value[thumbnails.value.length - 1];

    if(!lastThumbnail)
      return;

    const inset = `offset${props.columns ? 'Top' : 'Left'}`;
    const size = `offset${props.columns ? 'Height' : 'Width'}`;
    const lastThumbnailReached = [ ...thumbnails.value ].find(thumbnail => thumbnail[inset] >= lastThumbnail[inset] + lastThumbnail[size] - entries[0].target[size]);

    previewCorkSize.value = lastThumbnailReached[inset] - (lastThumbnail[inset] + lastThumbnail[size] - entries[0].target[size]);
  }
);
const scrollerResizeObserver = new ResizeObserver(
  entries => {
    scrollerBlockSize.value = entries[0].target.offsetHeight;

    const lastItem = items.value[items.value.length - 1];

    if(!lastItem)
      return;

    if(props.cork)
      scrollerCorkSize.value = entries[0].target.offsetWidth - lastItem.offsetWidth;
    else {
      const lastItemReached = [ ...items.value ].find(item => item.offsetLeft >= lastItem.offsetLeft + lastItem.offsetWidth - entries[0].target.offsetWidth);

      scrollerCorkSize.value = lastItemReached.offsetLeft - (lastItem.offsetLeft + lastItem.offsetWidth - entries[0].target.offsetWidth);
    }
  }
);

const nextItem = () => {
  preventIndexDetection.value = true;

  if(activeItemIndex.value == items.value.length - 1)
    return void (activeItemIndex.value = 0);

  if(!props.cork && items.value[activeItemIndex.value + 1].offsetLeft - (scroller.value.scrollWidth - scroller.value.offsetWidth) > 1)
    return void (activeItemIndex.value = 0);

  return void activeItemIndex.value++;
};

const previousItem = () => {
  preventIndexDetection.value = true;

  if(activeItemIndex.value == 0) {
    if(props.cork)
      return void (activeItemIndex.value = items.value.length - 1);

      return void (activeItemIndex.value = [ ...items.value.entries() ].reverse().find(([,item]) => item.offsetLeft - (scroller.value.scrollWidth - scroller.value.offsetWidth) <= 1)?.[0] ?? 0);
  }

  return void activeItemIndex.value--;
};

const resumeAutoRotation = debounce(
  () => {
    if(scrollerHovered.value || nextButtonFocused.value || nextButtonHovered.value || previewHovered.value || previousButtonFocused.value || previousButtonHovered.value)
      return;

    nextItem();

    autoRotating.value = true;
    startAutoRotation();
  },
  1000
);

const startAutoRotation = debounce(
  () => {
    if(scrollerHovered.value || nextButtonFocused.value || nextButtonHovered.value || previewHovered.value || previousButtonFocused.value || previousButtonHovered.value || !props.autoRotate || !autoRotating.value) {
      autoRotating.value = false;
      return;
    }

    nextItem();
  },
  props.autoRotateTiming < 1000 ? 1000 : props.autoRotateTiming
);

const nextButton_onblur = () => nextButtonFocused.value = false;

const nextButton_onclick = () => nextItem();

const nextButton_onfocus = () => nextButtonFocused.value = false;

const nextButton_onmouseleave = () => nextButtonHovered.value = false;

const nextButton_onmouseover = () => nextButtonHovered.value = true;

const preview_onclick = event => {
  if(event.target.parentElement == preview.value) {
    const index = [ ...thumbnails.value ].findIndex(thumbnail => thumbnail == event.target);

    if(index >= 0)
      activeItemIndex.value = index;
  }
};

const preview_onmouseleave = () => previewHovered.value = false;

const preview_onmouseover = () => previewHovered.value = true;

const previousButton_onblur = () => previousButtonFocused.value = false;

const previousButton_onclick = () => previousItem();

const previousButton_onfocus = () => previousButtonFocused.value = false;

const previousButton_onmouseleave = () => previousButtonHovered.value = false;

const previousButton_onmouseover = () => previousButtonHovered.value = true;

const scroller_onmouseleave = () => scrollerHovered.value = false;

const scroller_onmousemove = throttle(event => scrollerMousePositionX.value = event.clientX - scroller.value.getBoundingClientRect().left, 50);

const scroller_onmouseover = () => scrollerHovered.value = true;

const scroller_onscroll = debounce(
  () => {
    if(autoRotating.value)
      startAutoRotation();

    if(preventIndexDetection.value) {
      preventIndexDetection.value = false;
      return;
    }

    for(const index in items.value) {
      const item = items.value[index];

      if(item.offsetLeft == scroller.value.scrollLeft) {
        const indexAsNumber = Number(index);

        activeItemIndex.value = indexAsNumber;
        break;
      }
    }
  },
  50
);

const scroller_ontouch = () => preventIndexDetection.value = false;

const scroller_onwheel = () => preventIndexDetection.value = false;

watch(
  activeItemIndex,
  (current, previous) => {
    if(!scroller.value)
      return;

    if(current != previous) {
      scroller.value.scrollTo(
        {
          behavior: 'smooth',
          left: items.value[current]?.offsetLeft ?? 0
        }
      );

      if(!preview.value)
        return;

      thumbnails.value[previous]?.removeAttribute('data-active');
      thumbnails.value[current]?.setAttribute('data-active', '');

      preview.value.scrollTo(
        {
          behavior: 'smooth',
          [props.columns ? 'top' : 'left']: thumbnails.value[current]?.[`offset${props.columns ? 'Top' : 'Left'}`] ?? 0
        }
      );
    }
  }
);

watch(
  [ items, () => props.cork ],
  ([itemsCurrent, corkCurrent]) => {
    if(!scroller.value)
      return;

    const lastItem = itemsCurrent[itemsCurrent.length - 1];

    if(!lastItem)
      return;

    requestAnimationFrame(
      () => {
        if(corkCurrent)
          scrollerCorkSize.value = scroller.value.offsetWidth - lastItem.offsetWidth;
        else {
          const lastItemReached = [ ...itemsCurrent ].find(item => item.offsetLeft >= lastItem.offsetLeft + lastItem.offsetWidth - scroller.value.offsetWidth);

          scrollerCorkSize.value = lastItemReached.offsetLeft - (lastItem.offsetLeft + lastItem.offsetWidth - scroller.value.offsetWidth);
        }
      }
    );
  },
  { deep: true }
);

watch(
  thumbnails,
  current => {
    if(!preview.value)
      return;

    const lastThumbnail = current[current.length - 1];

    if(!lastThumbnail)
      return;

    requestAnimationFrame(
      () => {
        const inset = `offset${props.columns ? 'Top' : 'Left'}`;
        const size = `offset${props.columns ? 'Height' : 'Width'}`;
        const lastThumbnailReached = [ ...current ].find(thumbnail => thumbnail[inset] >= lastThumbnail[inset] + lastThumbnail[size] - preview.value[size]);

        previewCorkSize.value = lastThumbnailReached[inset] - (lastThumbnail[inset] + lastThumbnail[size] - preview.value[size]);
      }
    );
  },
  { deep: true }
);

watch(
  [
    scrollerHovered,
    nextButtonFocused,
    nextButtonHovered,
    previewHovered,
    previousButtonFocused,
    previousButtonHovered,
    autoRotating,
    () => props.autoRotate
  ],
  () => {
    if(props.autoRotate && !autoRotating.value)
      resumeAutoRotation();
  }
);

onMounted(
  () => {
    previewResizeObserver.observe(preview.value);
    scrollerResizeObserver.observe(scroller.value);

    if(props.autoRotate) {
      autoRotating.value = true;
      startAutoRotation();
    }

    thumbnails.value[activeItemIndex.value]?.setAttribute('data-active', '');
  }
);
</script>

<template>
<div
  data-component="Carousel"
  :data-columns="props.columns || undefined"
  :data-next-button-focused="nextButtonFocused || undefined"
  :data-next-button-hovered="nextButtonHovered || undefined"
  :data-next-button-visible="nextButtonHovered || nextButtonFocused || scrollerMousePositionX >= (scroller?.offsetWidth ?? 0) / 2 && scrollerHovered || undefined"
  :data-preview="props.preview || undefined"
  :data-preview-hovered="previewHovered || undefined"
  :data-previous-button-focused="previousButtonFocused || undefined"
  :data-previous-button-hovered="previousButtonHovered || undefined"
  :data-previous-button-visible="previousButtonHovered || previousButtonFocused || scrollerMousePositionX < (scroller?.offsetWidth ?? 0) / 2 && scrollerHovered || undefined"
  :data-scroller-hovered="scrollerHovered || undefined"
  ref="root"
  :style="{
    '--Carousel-preview-cork-size': `${previewCorkSize}px`,
    '--Carousel-scroller-block-size': `${scrollerBlockSize}px`,
    '--Carousel-scroller-cork-size': `${scrollerCorkSize}px`
  }"
>
  <div class="controls">
    <Button
      class="controls__button controls__button--previous"
      @blur="previousButton_onblur"
      @click="previousButton_onclick"
      @focus="previousButton_onfocus"
      @mouseleave="previousButton_onmouseleave"
      @mouseover="previousButton_onmouseover"
    >
      <span class="feather" :class="`feather-chevron${activeItemIndex == 0 ? 's-right' : '-left'}`"></span>
    </Button>

    <Button
      class="controls__button controls__button--next"
      @blur="nextButton_onblur"
      @click="nextButton_onclick"
      @focus="nextButton_onfocus"
      @mouseleave="nextButton_onmouseleave"
      @mouseover="nextButton_onmouseover"
    >
      <span class="feather" :class="`feather-chevron${activeItemIndex == items.length - 1 ? 's-left' : '-right' }`"></span>
    </Button>
  </div>

  <div
    class="scroller"
    ref="scroller"
    @mouseleave="scroller_onmouseleave"
    @mousemove="scroller_onmousemove"
    @mouseover="scroller_onmouseover"
    @scroll="scroller_onscroll"
    @touch="scroller_ontouch"
    @wheel="scroller_onwheel"
  >
    <slot/>

    <div class="scroller__cork"></div>
  </div>

  <div
    class="preview"
    ref="preview"
    @click="preview_onclick"
    @mouseleave="preview_onmouseleave"
    @mouseover="preview_onmouseover"
    v-if="props.preview"
  >
    <slot name="preview"/>

    <div class="preview__cork"></div>
  </div>
</div>
</template>

<style lang="scss">
[data-component="Carousel"] {
  .preview,
  .scroller {
    & > * {
      display: inline-block;
      scroll-snap-align: start;
    }
  }

  .preview {
    & > * {
      block-size: 100%;
      inline-size: auto;
      max-inline-size: 100%;
      transition-property: outline;

      &:not(:last-child, :nth-last-child(2)) {
        margin-inline-end: 1em;
      }

      &[data-active] {
        outline-color: currentColor;
        outline-offset: -2px;
        outline-style: solid;
        outline-width: 2px;
      }
    }
  }

  &[data-columns] {
    .preview {
      & > * {
        block-size: auto;
        inline-size: 100%;
        margin-inline-end: 0;
        max-block-size: 100%;
        max-inline-size: auto;
        
        &:not(:last-child, :nth-last-child(2)) {
          margin-block-end: 1em;
        }
      }
    }
  }
}
</style>

<style lang="scss" scoped>
[data-component="Carousel"] {
  block-size: 100%;
  display: grid;
  grid-template-columns: [ main-start ] 1fr [ main-end ];
  grid-template-rows: [ scroller-start ] auto [scroller-end];
  inline-size: 100%;

  .controls {
    align-items: center;
    display: flex;
    grid-area: scroller / main;
    justify-content: space-between;
    pointer-events: none;
    z-index: 1;

    & > * {
      pointer-events: initial;
    }
  }

  .controls__button {
    --_transition-property: opacity;

    opacity: 0;
  }

  .controls__button--previous {
    margin-inline-start: 1.5em;
  }

  .controls__button--next {
    margin-inline-end: 1.5em;
  }

  .scroller {
    grid-area: scroller / main;
    overflow: auto hidden;
    scrollbar-width: none;
    scroll-snap-type: x mandatory;
    white-space: nowrap;
    -ms-overflow-style: none;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  .scroller__cork {
    block-size: 1px;
    inline-size: var(--Carousel-scroller-cork-size);
  }

  .preview {
    block-size: 100%;
    grid-area: preview / main;
    inline-size: 100%;
    overflow: auto hidden;
    scrollbar-width: none;
    scroll-snap-type: x mandatory;
    white-space: nowrap;
    -ms-overflow-style: none;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  .preview__cork {
    block-size: 1px;
    inline-size: var(--Carousel-preview-cork-size);
  }

  &[data-next-button-visible] {
    .controls__button--next {
      opacity: 1;
    }
  }

  &[data-previous-button-visible] {
    .controls__button--previous {
      opacity: 1;
    }
  }

  &[data-preview] {
    grid-template-rows: [scroller-start] auto [scroller-end] 1em [preview-start] 7.5em [preview-end];
  }

  &[data-columns] {
    grid-template-columns: [ scroller-start ] auto [ scroller-end ];
    grid-template-rows: [ main-start ] 1fr [main-end];

    .controls {
      grid-area: main / scroller;
    }

    .scroller {
      grid-area: main / scroller;
    }

    .preview {
      block-size: var(--Carousel-scroller-block-size);
      grid-area: main / preview;
      white-space: normal;
      overflow: hidden auto;
      scroll-snap-type: y mandatory;
    }

    .preview__cork {
      block-size: var(--Carousel-preview-cork-size);
      inline-size: 1px;
    }

    &[data-preview] {
      grid-template-columns: [preview-start] 7.5em [preview-end] 1em [scroller-start] auto [scroller-end];
    }
  }
}
</style>
