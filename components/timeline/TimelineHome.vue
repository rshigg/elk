<script setup lang="ts">
const paginator = useMastoClient().v1.timelines.listHome({ limit: 30 })
const stream = $(useStreaming(client => client.v1.stream.streamUser()))

const isPWA: boolean = process.server ? false : useNuxtApp().$pwa.isInstalled

if (isHydrated && isPWA) {
  document.documentElement.style.setProperty('overscroll-behavior-y', 'none')
  document.body.style.setProperty('overscroll-behavior-y', 'none')
}

const refreshContainerMaxHeight = 80
const pullYStart = ref(0)
const deltaPullY = ref(0)
const refreshing = ref<boolean>(false)

function handleTouchStart(e: TouchEvent) {
  if (isPWA && window.scrollY === 0)
    pullYStart.value = e.touches[0].screenY
}

function handleTouchMove(e: TouchEvent) {
  const refreshContainer = document.getElementById('refresh-container')
  if (isPWA && window.scrollY === 0 && refreshContainer) {
    refreshContainer.style.removeProperty('transition')

    deltaPullY.value = (e.touches[0].screenY - pullYStart.value) * 0.35
    const refreshContainerHeight = Math.min(Math.max(deltaPullY.value, 0), refreshContainerMaxHeight)
    refreshContainer.style.height = `${refreshContainerHeight}px`
    // refreshContainerHeight === refreshContainerMaxHeight
    //   ? refreshContainer.style.setProperty('border-bottom', '2px solid #FFFFFF75')
    //   : refreshContainer.style.removeProperty('border-bottom')
  }
}

function closeRefreshContainer() {
  const refreshContainer = document.getElementById('refresh-container')
  refreshContainer!.style.setProperty('transition', 'height 300ms ease-in')
  refreshContainer!.style.height = '0px'
  setTimeout(() => {
    refreshContainer!.style.removeProperty('transition')
  }, 300)
}

function handleTouchEnd() {
  const refreshContainer = document.getElementById('refresh-container')
  if (isPWA && window.scrollY === 0 && refreshContainer) {
    if (deltaPullY.value >= refreshContainerMaxHeight) {
      // perform a refresh if the refresh container is fully expanded
      refreshing.value = true
      // window.location.reload()
    }
    closeRefreshContainer()
  }
}
</script>

<template>
  <div id="timeline-wrapper">
    <PublishWidget draft-key="home" border="b base" />
    <div v-if="isPWA" id="refresh-container" relative flex justify-center items-end overflow-hidden>
      <img src="/forest.webp" width="600" height="240" alt="" loading="lazy" absolute="~ top-1/4" opacity-50>
      <svg xmlns="http://www.w3.org/2000/svg" width="auto" height="90%" fill="none" viewBox="0 0 250 250" z-1 :class="{ 'animate-pulse': refreshing }">
        <path fill="#EA9E44" d="M131.7 48.2c-8.9 1-13.5-1.3-16.4 7.1 0 0 9.5 14.3 34.8 14-1.4 5.8-.8 10.1-.8 15.3 0 17.2-13.6 32.2-43.6 32.2-15.2 0-33.8 2.8-47.9 10.5a42.5 42.5 0 0 0-21.4 24.5l4 5.6h7.5v31.7l-9.2 14.5 2.7 42H50l.8-33.8a91.6 91.6 0 0 0 22.7-19.5 54.4 54.4 0 0 0 10-58.6l7.7-3.3c8.5 20.3 5.6 38-1.9 52.1a284 284 0 0 0 37-3l-2.4-29 8.3-.7 8 95.8h8.7l1.4-62.3c7-3 22.7-14 30-51.9a247.5 247.5 0 0 0 3.7-40.7l-12.3-4h38l3.7-8.4c-3.6.2-8-1.7-8-1.7l1-4.1h8l-43.2-22.7a50 50 0 0 0-13.4 7.2 48 48 0 0 0-26.2-8.8Z" />
        <path fill="#A75C26" d="M165.2 185.5c4.5-5 9.8-11.8 14.7-21.1.1 3.8.4 6.1.6 7.3l22.3 19.2L186 224l-7.6-3.6 10.5-24.3-23.7-10.7ZM84 191.2a387.4 387.4 0 0 0 18.2-.8c-7.4 11-16.2 18.8-20 21.9v33.3h-8.5L69.4 207a96.2 96.2 0 0 0 14.7-16ZM58.4 6.2 66.5 4c4 14.3 8.9 22 15 26 5.6 3.4 13.1 3.5 21 3a23 23 0 0 1-6.4-3.3c-4.5-3.3-8.4-8.9-11.2-18.9l8-2.4c2.2 7.6 4.7 12 8.1 14.4a24 24 0 0 0 12.3 3.6c9 1 20.4 1.5 34.6 8.1 2 .8 4 1.7 5.8 2.7a16.8 16.8 0 0 0-.8-1.6 31.3 31.3 0 0 0-16-11.2l2.8-8C154 21.6 159.7 29 162 35c2.7 6.7 1 12 1 12-2 .6-4 1.6-6.3 2.3-4.4-3-8.5-5.3-12.5-7.1-14.5-5.5-28.7-1.8-41.3-.6-9.5.9-18.2.5-25.7-4.3-7.4-4.8-14-13.8-18.7-31Z" />
        <path fill="#A75C26" d="M114 20a29 29 0 0 1-2.3-14.2l8.4.6c-.5 7 1.9 11.7 4.6 14.8-3.5-.7-6.9-.7-10.8-1.2Zm52.9-.9 5.5-6.4c14.3 12.6 5.4 30.7 5.4 30.7l-7.8-1c-.3-4-.8-11.7-4.4-16 2.3 1.2 4.4 2.5 6.6 4.5 0-3.7-1.2-8.1-5.3-11.8Z" />
      </svg>
    </div>
    <TimelinePaginator v-bind="{ paginator, stream }" :preprocess="reorderedTimeline" context="home" @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd" />
  </div>
</template>
