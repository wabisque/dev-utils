<script setup>
import { computed, ref, watch } from 'vue';

const emits = defineEmits([ 'update:modelValue' ]);
const props = defineProps(
  {
    interactive: {
      type: Boolean,
      default: false
    },
    maximum: {
      type: Number,
      default: 5
    },
    modelValue: {
      type: Number,
      default: 0,
    }
  }
);

const hovered = ref(false);
const hoveredBackgroundStarIndex = ref(0);

const interactiveCurrent = computed(() => Math.floor(props.modelValue));
const percentage = computed(
  () => {
    const current = props.interactive ? interactiveCurrent.value : props.modelValue;

    return (current > props.maximum ? 1 : (current < 0 ? 0 : (current / props.maximum) ?? 0)) * 100;
  }
);

const imageBackgroundStar_onclick = index => {
  if(props.interactive)
    emits('update:modelValue', index == interactiveCurrent.value ? 0 : index);
};

const image_onmouseleave = () => hovered.value = false;

const image_onmouseover = () => hovered.value = true;

const imageBackgroundStar_onmouseleave = () => hoveredBackgroundStarIndex.value = 0;

const imageBackgroundStar_onmouseover = index => hoveredBackgroundStarIndex.value = index;

watch(
  interactiveCurrent,
  value => {
    if(props.interactive && value != props.modelValue)
      emits('update:modelValue', value);
  }
);
</script>

<template>
<div
  data-component="StarRating"
  :data-hovered="props.interactive && hovered|| undefined"
  :data-interactive="props.interactive || undefined"
>
  <svg
    class="image"
    height="24"
    preseveAspectRatio="xMinYmin meet"
    :width="24 * (props.maximum || 1)"
    :viewBox="`0 0 ${24 * (props.maximum || 1)} 24`"
    @mouseleave="image_onmouseleave"
    @mouseover="image_onmouseover"
  >
    <mask id="StarRating-component-svg-mask-stars">
      <g fill="white">
        <use
          href="#StarRating-component-svg-symbol-star"
          width="24"
          :x="(index - 1) * 24"
          y="0"
          v-for="index in (maximum || 1)"
        ></use>
      </g>
    </mask>

    <g class="image__background">
      <use
        :data-active="hoveredBackgroundStarIndex > 0 && index <= hoveredBackgroundStarIndex || undefined"
        class="image__background__star"
        href="#StarRating-component-svg-symbol-star"
        width="24"
        :x="(index - 1) * 24"
        y="0"
        @click="imageBackgroundStar_onclick(index)"
        @mouseleave="imageBackgroundStar_onmouseleave"
        @mouseover="imageBackgroundStar_onmouseover(index)"
        v-for="index in (maximum || 1)"
      ></use>
    </g>

    <rect
      class="image__foreground"
      height="100%"
      style="mask: url(#StarRating-component-svg-mask-stars)"
      :width="`${percentage}%`"
      x="0"
      y="0"
    ></rect>
  </svg>

  <svg style="display: none;">
    <symbol id="StarRating-component-svg-symbol-star" viewBox="0 0 24 24">
      <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
    </symbol>
  </svg>
</div>
</template>

<style lang="scss">
</style>

<style lang="scss" scoped>
[data-component="StarRating"] {
  --_background-color-light: white;
  --_background-color-dark: black;
  --_background-color: var(--_background-color-light);

  --_color-light: #0088ff;
  --_color-dark: #0088ff;
  --_color: var(--_color-light);

  --_highlight-color-light: #ff880044;
  --_highlight-color-dark: #ff880044;
  --_highlight-color: var(--_highlight-color-light);

  @media (prefers-color-scheme: dark) {
    --_background-color: var(--_background-color-dark);
    --_color: var(--_color-dark);
    --_highlight-color: var(--_highlight-color-dark);
  }

  &[color-scheme="light"],
  [color-scheme="light"] {
    --_background-color: var(--_background-color-light);
    --_color: var(--_color-light);
    --_highlight-color: var(--_highlight-color-light);
  }

  &[color-scheme="dark"],
  [color-scheme="dark"] {
    --_background-color: var(--_background-color-dark);
    --_color: var(--_color-dark);
    --_highlight-color: var(--_highlight-color-dark);
  }

  display: inline-block;

  .image {
    block-size: auto;
    inline-size: 100%;
  }

  .image__background {
    fill: var(--_background-color);
    stroke: var(--_color);
    stroke-width: 1px;
  }

  .image__foreground {
    fill: var(--_color);
    pointer-events: none;
    transition-property: fill, opacity, stroke, width;
  }

  .image__background__star {
    transition-property: fill, opacity, stroke;
  }

  &[data-interactive] {
    &[data-hovered] {
      .image__background__star {
        opacity: 0.5;

        &[data-active] {
          fill: var(--_highlight-color);
          opacity: 1;
        }
      }

      .image__foreground {
        opacity: 0.75;
      }
    }
  }
}
</style>