<template>
  <tr :class="['entry-container', { even: index % 2 === 0}]">
    <td class="desc">
      {{ entry.desc }}
    </td>
    <td class="from">
      {{ entry.from.hours }}:{{ entry.from.minutes }}
    </td>
    <td class="to">
      {{ entry.to.hours }}:{{ entry.to.minutes }}
    </td>
    <td class="hrs">
      {{ entry.hrs }}
    </td>
    <td class="rate">
      {{ entry.rate }}
    </td>
    <td class="total">
      {{ entry.total }}
    </td>
    <td class="controls">
      <button class="danger" @click="$emit('delete-entry')">Delete</button>
    </td>
  </tr>
</template>

<script>
export default {
  emits: ['delete-entry'],
  props: {
    entry: {},
    index: {}
  }
}
</script>

<style scoped>
  .entry-container {
    border: 1px solid #2f2f2f;
    &.even {
      background: #2f2f2f;
    }
    > td {
      padding: 10px;
      border-right: 1px solid #2f2f2f;
      text-align: center;
      &:last-child {
        border-right: none;
      }
      &.desc {
        text-align: left;
      }
      &.controls {
        width: 50px;
      }
    }

  }
</style>