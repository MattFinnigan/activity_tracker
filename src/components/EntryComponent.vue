<template>
  <div :class="['entry-container', { even: index % 2 === 0}]">
    <div class="desc">
      {{ entry.desc }}
    </div>
    <div class="from">
      {{ entry.from }}
    </div>
    <div class="to">
      {{ entry.to }}
    </div>
    <div class="hrs">
      {{ entry.hrs }}
    </div>
    <div class="rate">
      {{ entry.rate }}
    </div>
    <div class="total">
      {{ entry.total }}
    </div>
  </div>
</template>

<script>
export default {
  props: {
    entry: {},
    index: {}
  }
}
</script>

<style scoped>
  .entry-container {
    border: 1px solid grey;
    &.even {
      background: grey;
    }
    > div {
      padding: 10px;
      border-right: 1px solid grey;
      &:last-child {
        border-right: none;
      }
    }
    
  }
</style>