<template>
  <div class="h-full flex flex-col w-screen md:w-96 xl:w-128">
    <div
      class="
        py-3 px-6 flex flex-none border-b border-gray-200
        dark:border-dark-200
      "
    >
      <div>
        <NavPlaceholder class="md:hidden" />
        <div class="text-lg">
          Bag
        </div>
        <div class="opacity-50 text-xs">
          {{ bags.length }} icons picked
        </div>
      </div>
      <div class="flex-auto" />
      <IconButton v-if="bags.length" class="text-xl mr-4 flex-none" icon="carbon:delete" @click="clear" />
      <IconButton class="text-2xl flex-none" icon="carbon:close" @click="$emit('close')" />
    </div>

    <template v-if="bags.length">
      <div class="flex-auto overflow-y-auto py-3 px-1">
        <Icons :icons="bags" @select="e => $emit('select', e)" />
      </div>
      <div
        class="
          flex-none border-t border-gray-200 py-3 px-6 text-2xl opacity-75
          dark:border-dark-200
        "
      >
        <IconButton class="p-1 cursor-pointer hover:text-primary" icon="carbon:function" text="Generate Icon Fonts" :active="true" @click="packIconFont" />
        <IconButton class="p-1 cursor-pointer hover:text-primary" icon="carbon:download" text="Download SVGs Zip" :active="true" @click="packSvgs" />
      </div>
    </template>

    <template v-else>
      <div class="text-center px-4 py-8 text-gray-500 italic font-light text-sm">
        No icons yet ;)
      </div>
    </template>
  </div>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
import { bags, clearBag } from '../store'
import { PackIconFont, PackSvgZip } from '../utils/pack'

export default defineComponent({
  setup(props, { emit }) {
    const clear = () => {
      // eslint-disable-next-line no-alert
      if (confirm('Are you sure to remove all icons from the bag?')) {
        clearBag()
        emit('close')
      }
    }

    const packIconFont = async() => {
      // TODO: customzie
      await PackIconFont(
        bags.value,
      )
    }

    const packSvgs = async() => {
      // TODO: customzie
      await PackSvgZip(
        bags.value.map(i => i.replace(':', '-')),
        'icones-bags',
      )
    }

    return {
      clear,
      bags,
      packIconFont,
      packSvgs,
    }
  },
})
</script>
