<template>
  <WithNavbar>
    <div class="flex flex-auto h-full overflow-hidden ">
      <Drawer class="h-full overflow-auto flex-none hidden md:block" style="width:280px" />
      <div v-if="collection" class="py-5 h-full overflow-y-auto flex-auto overflow-x-hidden relative">
        <!-- Loading -->
        <div
          class="absolute top-0 left-0 right-0 bottom-0 bg-white bg-opacity-75 content-center transition-opacity duration-100 z-50 dark:bg-dark-100"
          :class="loading ? '' : 'opacity-0 pointer-events-none'"
        >
          <div class="absolute text-gray-800 dark:text-dark-500" style="top:50%;left:50%;transform:translate(-50%,-50%)">Loading...</div>
        </div>

        <div class="flex px-8">
          <!-- Left -->
          <div class="flex-auto px-2">
            <NavPlaceholder class="md:hidden" />

            <div class="text-gray-900 text-xl flex select-none dark:text-gray-200">
              <div class="whitespace-no-wrap overflow-hidden">
                {{ collection.name }}
              </div>
              <a
                v-if="collection.url"
                class="hover:text-gray-900 ml-1 mt-1 text-base opacity-25"
                :href="collection.url"
                target="_blank"
              >
                <Icon icon="la:external-link-square-alt-solid" />
              </a>
              <div class="flex-auto" />
            </div>
            <div class="text-xs block opacity-50">
              {{ collection.author }}
            </div>
            <div>
              <a
                class="text-xs opacity-50 hover:opacity-100"
                :href="collection.licenseURL"
                target="_blank"
              >{{ collection.license }}</a>
            </div>
          </div>

          <!-- Right -->
          <div class="flex flex-col">
            <ActionsMenu :collection="collection" />
            <div class="flex-auto" />
          </div>
        </div>

        <!-- Categories -->
        <div class="py-2 px-7 overflow-x-auto flex flex-no-wrap select-none">
          <template v-if="collection.categories">
            <div
              v-for="c of Object.keys(collection.categories)"
              :key="c"
              class="
                whitespace-no-wrap text-sm inline-block px-2 border border-gray-200 rounded-full m-1 hover:bg-gray-100 cursor-pointer
                dark:border-dark-200 dark-hover:bg-dark-200
              "
              :class="c === category ? 'text-primary border-primary dark:border-primary' : 'opacity-75'"
              @click="toggleCategory(c)"
            >
              {{ c }}
            </div>
          </template>
        </div>

        <!-- Searching -->
        <div
          class="
            mx-8 my-2 hidden md:flex shadow rounded outline-none py-1 px-4
            border border-transparent dark:border-dark-200
          "
        >
          <Icon icon="carbon:search" class="m-auto flex-none opacity-60" />
          <input
            ref="input"
            v-model="search"
            class="text-base outline-none py-1 px-4 flex-auto m-0 bg-transparent"
            placeholder="Search..."
          >
          <Icon v-if="search" icon="carbon:close" class="m-auto text-lg -mr-1 opacity-60" @click="search = ''" />
        </div>

        <!-- Icons -->
        <div class="px-4 pt-2 pb-4 text-center">
          <Icons
            :icons="icons.slice(0, max)"
            :selected="bags"
            :size="iconSize"
            :display="listType"
            :search="search"
            :namespace="namespace"
            @select="onSelect"
          />
          <button v-if="icons.length > max" class="btn m-2" @click="loadMore">
            Load More
          </button>
          <p class="text-gray-500 text-sm pt-4">
            {{ icons.length }} icons
          </p>
        </div>

        <Footer />

        <!-- Bag Fab -->
        <FAB
          v-if="bags.length"
          icon="carbon:shopping-bag"
          :number="bags.length"
          @click="showBag = true"
        />

        <!-- Bag -->
        <Modal :value="showBag" direction="right" @close="showBag = false">
          <Bag
            @close="showBag = false"
            @select="e => current = e"
          />
        </Modal>

        <!-- Details -->
        <Modal :value="!!current" @close="current = ''">
          <IconDetail :icon="current" :show-collection="collection.id === 'all'" @close="current = ''" />
        </Modal>

        <!-- Selecting Note -->
        <div
          class="fixed top-0 right-0 pl-4 pr-2 py-1 rounded-l-full bg-primary text-white shadow mt-16 cursor-pointer transition-transform duration-300 ease-in-out"
          :style="selectingMode ? {} : {transform: 'translateX(120%)'}"
          @click="selectingMode = false"
        >
          Selecting Mode
          <Icon icon="carbon:close" class="inline-block text-xl align-text-bottom" />
        </div>

        <SearchElectron />
      </div>
    </div>
  </WithNavbar>
</template>

<script lang='ts'>
import { defineComponent, ref, computed } from 'vue'
import { iconSize, listType, selectingMode, bags, toggleBag, getSearchResults, isCurrentCollectionLoading } from '../store'
import { isElectron } from '../env'

export default defineComponent({
  setup(props) {
    const { search, icons, category, collection } = getSearchResults()
    const showBag = ref(false)

    const current = ref<string>('')
    const max = ref(isElectron ? 500 : 200)

    const toggleCategory = (cat: string) => {
      if (category.value === cat) category.value = ''
      else category.value = cat
    }

    const namespace = computed(() => {
      return !collection.value || collection.value.id === 'all'
        ? ''
        : `${collection.value.id}:`
    })

    const onSelect = (icon: string) => {
      if (selectingMode.value) toggleBag(icon)
      else current.value = icon
    }

    const selectedIcons = computed(() => {
      if (selectingMode.value) return bags.value
      else return current.value ? [] : [current.value]
    })

    const loadMore = () => {
      max.value += 100
    }

    return {
      loading: isCurrentCollectionLoading(),
      current,
      search,
      collection,
      icons,
      onSelect,
      max,
      category,
      loadMore,
      iconSize,
      listType,
      namespace,
      selectedIcons,

      // cates
      toggleCategory,

      // bags
      showBag,
      bags,
      selectingMode,
    }
  },
})
</script>
