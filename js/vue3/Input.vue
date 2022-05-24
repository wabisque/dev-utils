<script>
export default { inheritAttrs: false };
</script>

<script setup>
import { computed, ref, useAttrs } from 'vue';

const props = defineProps(
  {
    icon: {
      type: [ Array, Object, String ],
      default: '',
    }
  }
);
const attrs = useAttrs();

const inputAttrs = computed(() => Object.fromEntries(Object.entries(attrs).filter(entry => entry[0] != 'class')));

const focused = ref(false);
</script>

<template>
<div
  data-component="Input"
  :data-focused="focused || undefined"
  :data-icon="!!props.icon || undefined"
  v-bind="{ class: attrs.class }"
>
  <select class="input" v-bind="inputAttrs" v-if="inputAttrs.type == 'select'">
    <slot/>
  </select>
  <input class="input" v-bind="inputAttrs" v-else>

  <span
    class="icon"
    :class="props.icon"
    v-if="!!icon"
  ></span>
</div>
</template>

<style lang="scss" scoped>
[data-component="Input"] {
  --_background-color-light: hsl(var(--theme-color-background-light));
  --_background-color-dark: hsl(var(--theme-color-background-dark));
  --_background-color: var(--_background-color-light);

  --_border-color-light: hsl(var(--theme-color-border-light));
  --_border-color-dark: hsl(var(--theme-color-border-dark));
  --_border-color: var(--_border-color-light);

  --_color-light: hsl(var(--theme-color-text-light));
  --_color-dark: hsl(var(--theme-color-text-dark));
  --_color: var(--_color-light);

  --_icon-color-light: hsl(var(--theme-color-text-light) / 0.5);
  --_icon-color-dark: hsl(var(--theme-color-text-dark) / 0.5);
  --_icon-color: var(--_icon-color-light);

  --focused_background-color-light: hsl(var(--theme-color-background-light));
  --focused_background-color-dark: hsl(var(--theme-color-background-dark));
  --focused_background-color: var(--focused_background-color-light);

  --focused_border-color-light: hsl(var(--theme-color-brand-light));
  --focused_border-color-dark: hsl(var(--theme-color-brand-dark));
  --focused_border-color: var(--focused_border-color-light);

  --focused_color-light: hsl(var(--theme-color-text-light));
  --focused_color-dark: hsl(var(--theme-color-text-dark));
  --focused_color: var(--focused_color-light);

  --focused_icon-color-light: hsl(var(--theme-color-text-light));
  --focused_icon-color-dark: hsl(var(--theme-color-text-dark));
  --focused_icon-color: var(--focused_icon-color-light);

  @media (prefers-color-scheme: dark) {
    --_background-color: var(--_background-color-dark);
    --_border-color: var(--_border-color-dark);
    --_color: var(--_color-dark);
    --_icon-color: var(--_icon-color-dark);

    --focused_background-color: var(--focused_background-color-dark);
    --focused_border-color: var(--focused_border-color-dark);
    --focused_color: var(--focused_color-dark);
    --focused_icon-color: var(--focused_icon-color-dark);
  }

  [color-scheme="light"] {
    --_background-color: var(--_background-color-light);
    --_border-color: var(--_border-color-light);
    --_color: var(--_color-light);
    --_icon-color: var(--_icon-color-light);

    --focused_background-color: var(--focused_background-color-light);
    --focused_border-color: var(--focused_border-color-light);
    --focused_color: var(--focused_color-light);
    --focused_icon-color: var(--focused_icon-color-light);
  }

  [color-scheme="dark"] {
    --_background-color: var(--_background-color-dark);
    --_border-color: var(--_border-color-dark);
    --_color: var(--_color-dark);
    --_icon-color: var(--_icon-color-dark);

    --focused_background-color: var(--focused_background-color-dark);
    --focused_border-color: var(--focused_border-color-dark);
    --focused_color: var(--focused_color-dark);
    --focused_icon-color: var(--focused_icon-color-dark);
  }

  display: inline-block;

  :where(.input) {
    background-color: var(--_background-color);
    border: 1px solid var(--_border-color);
    border-radius: var(--theme-border-radius-small);
    color: var(--_color);
    inline-size: 100%;
    padding-block: 0.65em;
    padding-inline: 0.85em;
    transition-property: background-color, border, border-radius, color;

    &:where(:focus) {
      background-color: var(--focused_background-color);
      border-color: var(--focused_border-color);
      color: var(--focused_color);

      & ~ :where(.icon) {
        color: var(--focused_icon-color);
      }
    }
  }

  &:where([data-icon]) {
    :where(.input) {
      padding-inline-end: 2em;
    }

    :where(.icon) {
      align-items: center;
      block-size: 100%;
      display: flex;
      color: var(--_icon-color);
      inset-block-start: 0;
      inset-inline-end: 0.5em;
      pointer-events: none;
      position: absolute;
      transition-property: color;
    }
  }

  &:where([data-focused]) {
    :where(.input) {
      background-color: var(--focused_background-color);
      border-color: var(--focused_border-color);
      color: var(--focused_color);
    }

    :where(.icon) {
      color: var(--focused_icon-color);
    }
  }
}
</style>
