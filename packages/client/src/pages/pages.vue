<script setup lang="ts">
import type { RouterInfo } from '@vue/devtools-kit'
import { VueInput } from '@vue/devtools-ui'
import { getMatchedRoutes, getRouterInfo, navigateAction, onRouterInfoUpdated } from '@vue/devtools-core'
import type { RouteLocationNormalizedLoaded, RouteRecordNormalized } from 'vue-router'

const routeInput = ref('')
const currentRoute = ref<RouteLocationNormalizedLoaded | null>(null)
const matchedRoutes = ref<RouteRecordNormalized[]>([])
const routeInputMatched = computed(() => {
  if (routeInput.value === currentRoute.value?.path)
    return []
  else
    return matchedRoutes.value
})

const routes = ref<RouteRecordNormalized[]>([])

function init(data: RouterInfo) {
  routes.value = data.routes
  currentRoute.value = data.currentRoute as RouteLocationNormalizedLoaded
  // router.value = data.router as Router
  routeInput.value = currentRoute.value?.path ?? '/'
}

function navigate() {
  if (routeInputMatched.value.length)
    navigateToRoute(routeInput.value)
}

function navigateToRoute(path: string) {
  navigateAction({
    path,
  })
}

onDevToolsClientConnected(() => {
  getRouterInfo().then((data) => {
    init(JSON.parse(data!))
  })
  onRouterInfoUpdated((data) => {
    init(data)
  })
})

watchDebounced(routeInput, () => {
  if (routeInput.value === currentRoute.value?.path)
    return
  getMatchedRoutes(routeInput.value).then((data) => {
    matchedRoutes.value = JSON.parse(data!)
  })
})
</script>

<template>
  <div block h-screen of-auto>
    <div h-full of-auto>
      <div border="b base" flex="~ col gap1" px4 py3>
        <div>
          <template v-if="false">
            <span op50>Navigate from </span>
            <span font-mono>Hi</span>
            <span op50> to </span>
          </template>
          <template v-else>
            <span op50>Current route</span>
          </template>
        </div>
        <VueInput
          v-model="routeInput"
          left-icon="i-carbon-direction-right-01 scale-y--100"
          :class="currentRoute?.path === routeInput ? '' : routeInputMatched.length ? 'text-green!' : 'text-orange!'"
          @keydown.enter="navigate"
        />
        <div>
          <template v-if="currentRoute?.path !== routeInput">
            <span>Press <b font-bold>Enter</b> to navigate</span>
            <span v-if="!routeInputMatched.length" text-orange op75> (no match)</span>
          </template>
          <template v-else>
            <span op50>Edit path above to navigate</span>
          </template>
        </div>
      </div>
      <!-- <SectionBlock
      icon="i-carbon-tree-view"
      text="Matched Routes"
      :padding="false"
    >
      <RoutesTable
        v-if="routeInputMatched.length"
        :pages="routeInputMatched"
        :matched="route.matched"
        :matched-pending="routeInputMatched"
        @navigate="navigateToRoute"
      />
    </SectionBlock> -->
      <SectionBlock
        icon="i-carbon-tree-view-alt"
        text="All Routes"
        :description="`${routes.length} routes registered in your application`"
        :padding="false"
      >
        <RoutesTable
          v-if="routes.length"
          :pages="routes"
          :matched="currentRoute?.matched ?? []"
          :matched-pending="routeInputMatched"
          @navigate="navigateToRoute"
        />
      </SectionBlock>
    </div>
  </div>
</template>
