<script setup>
import { ref } from 'vue';

const hovered = ref(false);
</script>

<template>
<button
  data-component="Button"
  :data-hovered="hovered || undefined"
  @mouseover="
    hovered = true;
  "
  @mouseleave="
    hovered = false;
  "
>
  <slot></slot>
</button>
</template>

<style lang="scss" scoped>
[data-component="Button"] {
  --_transition-property: ;

  --_background-color-light: white;
  --_background-color-dark: black;
  --_background-color: var(--_background-color-light);

  --_border-color-light: white;
  --_border-color-dark: black;
  --_border-color: var(--_border-color-light);

  --_box-shadow-color-light: #00000000;
  --_box-shadow-color-dark: #ffffff00;
  --_box-shadow-color: var(--_box-shadow-color-light);

  --_color-light: black;
  --_color-dark: white;
  --_color: var(--_color-light);

  --hovered_background-color-light: #0088ff;
  --hovered_background-color-dark: #0088ff;
  --hovered_background-color: var(--hovered_background-color-light);

  --hovered_border-color-light: #0088ff;
  --hovered_border-color-dark: #0088ff;
  --hovered_border-color: var(--hovered_border-color-light);

  --hovered_box-shadow-color-light: #00000033;
  --hovered_box-shadow-color-dark: #ffffff33;
  --hovered_box-shadow-color: var(--hovered_box-shadow-color-light);

  --hovered_color-light: white;
  --hovered_color-dark: white;
  --hovered_color: var(--hovered_color-light);

  @media (prefers-color-scheme: dark) {
    --_background-color: var(--_background-color-dark);
    --_border-color: var(--_border-color-dark);
    --_box-shadow-color: var(--_box-shadow-color-dark);
    --_color: var(--_color-dark);

    --hovered_background-color: var(--hovered_background-color-dark);
    --hovered_border-color: var(--hovered_border-color-dark);
    --hovered_box-shadow-color: var(--hovered_box-shadow-color-dark);
    --hovered_color: var(--hovered_color-dark);
  }

  &[color-scheme="light"],
  [color-scheme="light"] {
    --_background-color: var(--_background-color-light);
    --_border-color: var(--_border-color-light);
    --_box-shadow-color: var(--_box-shadow-color-light);
    --_color: var(--_color-light);

    --hovered_background-color: var(--hovered_background-color-light);
    --hovered_border-color: var(--hovered_border-color-light);
    --hovered_box-shadow-color: var(--hovered_box-shadow-color-light);
    --hovered_color: var(--hovered_color-light);
  }

  &[color-scheme="dark"],
  [color-scheme="dark"] {
    --_background-color: var(--_background-color-dark);
    --_border-color: var(--_border-color-dark);
    --_box-shadow-color: var(--_box-shadow-color-dark);
    --_color: var(--_color-dark);

    --hovered_background-color: var(--hovered_background-color-dark);
    --hovered_border-color: var(--hovered_border-color-dark);
    --hovered_box-shadow-color: var(--hovered_box-shadow-color-dark);
    --hovered_color: var(--hovered_color-dark);
  }

  background-color: var(--_background-color);
  border-radius: 0.25em;
  box-shadow: 0.1em 0.2em 0.25em 0.4em var(--_box-shadow-color);
  border: 1px solid var(--_border-color);
  color: var(--_color);
  cursor: pointer;
  padding-block: 0.65em;
  padding-inline: 0.85em;
  transition-property: background-color, border, border-radius, color var(--_transition-property);

  &:where([data-hovered], :hover) {
    background-color: var(--hovered_background-color);
    border-color: var(--hovered_border-color);
    box-shadow: 0.1em 0.2em 0.4em 0.25em var(--hovered_box-shadow-color);
    color: var(--hovered_color);
  }
}
</style>
