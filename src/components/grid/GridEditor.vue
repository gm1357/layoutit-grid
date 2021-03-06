<template>
  <section
    ref="sectionElement"
    :class="{ active: isCurrent, dragging }"
    :style="{
      gridTemplateRows,
      gridTemplateColumns,
      gridGap,
      display: 'grid',
    }"
    class="grid"
    @pointerdown="$refs.selection.cellDown($event)"
  >
    <AreaEditor v-for="a in areasToShow" :key="`area-${a.name}`" :area="a" @edit="$refs.selection.editArea(a)" />

    <GridCell
      v-for="(section, i) in gridSections(grid)"
      :key="`section-${i}`"
      :area="area"
      :section="section"
      :grid-computed-styles="gridComputedStyles"
      :grayed="!isActive"
      :focused="isFocused(section)"
      @togglelinename="$refs[$event].toggle()"
    >
      <LineName
        v-if="section.row.start === grid.row.sizes.length"
        :ref="`colLine-${section.col.start}`"
        :grid="grid"
        :pos="section.col.start"
        type="col"
      />

      <TrackSize
        v-if="isCurrent && section.row.start === grid.row.sizes.length"
        :grid="grid"
        type="col"
        :track="section.col.start"
      />

      <LineName
        v-if="section.col.start === grid.col.sizes.length"
        :ref="`rowLine-${section.row.start}`"
        :grid="grid"
        :pos="section.row.start"
        type="row"
      />

      <TrackSize
        v-if="isCurrent && section.col.start === grid.col.sizes.length"
        :grid="grid"
        type="row"
        :track="section.row.start"
      />
    </GridCell>

    <AreaSelection ref="selection" :area="area" @editstart="(a) => (editingArea = a)" @editend="editingArea = null" />
  </section>
</template>

<script setup="props, { el }">
export { default as GridCell } from './GridCell.vue'
export { default as AreaSelection } from './AreaSelection.vue'
export { default as LineName } from './LineName.vue'
export { default as TrackSize } from './TrackSize.vue'
export { default as AreaEditor } from '../area/AreaEditor.vue'

export { currentArea, dragging, currentFocus } from '../../store.js'
import { useIsCurrentArea, useIsActiveArea } from '../../composables/area.js'

export { gridSections } from '../../utils.js'

export default {
  props: {
    area: { type: Object, required: true },
  },
}

import { ref, computed, toRefs } from 'vue'
import { parseValue, parseUnit, parseValueUnit } from '../../store'

export const grid = computed(() => props.area.grid)

export const editingArea = ref(null)

export const areasToShow = computed(() => grid.value.areas.filter((a) => a !== editingArea.value))

function gridSizesForView(type) {
  return grid.value[type].sizes
    .map((size) => {
      const unit = parseUnit(size)
      switch (unit) {
        case 'auto':
          return '200px'
        case 'min-content':
          return '100px'
        case 'max-content':
          return '300px'
        default:
          return size
      }
    })
    .join(' ')
}

export const gridTemplateRows = computed(() => gridSizesForView('row'))
export const gridTemplateColumns = computed(() => gridSizesForView('col'))

const { area } = toRefs(props)
export const isCurrent = useIsCurrentArea(area)
export const isActive = useIsActiveArea(area)

export const sectionElement = ref(null)
export function gridComputedStyles() {
  return window.getComputedStyle(sectionElement.value)
}

function toViewGap(gap) {
  // Defaults to 1px so grid gap doesn't disappear
  return parseValue(gap) === 0 ? '1px' : gap
}

export const gridGap = computed(() => {
  return `${toViewGap(grid.value.row.gap)} ${toViewGap(grid.value.col.gap)}`
})

export function isFocused(section) {
  const tf = currentFocus.value
  return (
    tf &&
    tf.on === 'track' &&
    tf.grid === grid.value &&
    ((tf.type === 'row' && tf.track === section.row.start) || (tf.type === 'col' && tf.track === section.col.start))
  )
}
</script>

<style scoped lang="scss">
.grid {
  touch-action: none;
  height: 100%;
  position: relative;
  background: #300548;
  overflow: hidden;
  user-select: none;
  &.dragging section:hover {
    background: #fff;
  }
}
</style>
