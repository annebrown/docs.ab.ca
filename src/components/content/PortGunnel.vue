<script setup lang="ts">
import { computed } from 'vue'

const { data: navigation } = await useAsyncData('navigation', () => fetchContentNavigation())

const organizedNavigation = computed(() => {
  if (!navigation.value) return []
  
  const items = [...navigation.value]
  const index = items.find(item => item._path === '/index' || item._path === '/')
  
  if (index) {
    const indexIndex = items.indexOf(index)
    items.splice(indexIndex, 1)
    
    // Make the index item expandable and contain all other items
    index.children = items
    return [index]
  }
  
  return items
})

const uniqueRoutes = new Set()

const filterDuplicateRoutes = (items) => {
  return items.filter(item => {
    if (uniqueRoutes.has(item._path)) {
      return false
    }
    uniqueRoutes.add(item._path)
    if (item.children) {
      item.children = filterDuplicateRoutes(item.children)
    }
    return true
  })
}

const filteredNavigation = computed(() => filterDuplicateRoutes(organizedNavigation.value))
</script>

<template>
  <nav>
    <p class="text-xs m-0 p-0 font-bold">
        <NuxtLink href="/">Home</NuxtLink>
    </p>
    <ul class="navigation-tree my-0">
      <NaviNavigationItem
        v-for="item in filteredNavigation"
        :key="item._path"
        :item="item"
        :depth="0"
      />
    </ul>
    <p class="text-xs m-0 p-0 font-bold">
        <NuxtLink href="/about">About</NuxtLink>
    </p>

  </nav>
</template>

<style scoped>
.navigation-tree {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
</style>