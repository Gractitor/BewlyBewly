<script setup lang="ts">
import { useI18n } from 'vue-i18n'

import { settings } from '~/logic'
import api from '~/utils/api'

const emit = defineEmits<{
  (e: 'itemClick', item: { name: string, url: string, unreadCount: number, icon: string }): void
}>()

const { t } = useI18n()
const list = computed((): { name: string, url: string, unreadCount: number, icon: string }[] => [
  {
    name: t('topbar.noti_dropdown.replys'),
    url: 'https://message.bilibili.com/#/reply',
    unreadCount: 0,
    icon: 'i-solar:reply-2-bold-duotone',
  },
  {
    name: t('topbar.noti_dropdown.mentions'),
    url: 'https://message.bilibili.com/#/at',
    unreadCount: 0,
    icon: 'i-solar:mention-circle-bold-duotone',
  },
  {
    name: t('topbar.noti_dropdown.likes'),
    url: 'https://message.bilibili.com/#/love',
    unreadCount: 0,
    icon: 'i-solar:like-bold-duotone',
  },
  {
    name: t('topbar.noti_dropdown.messages'),
    url: 'https://message.bilibili.com/#/system',
    unreadCount: 0,
    icon: 'i-solar:chat-line-bold-duotone',
  },
  {
    name: t('topbar.noti_dropdown.chats'),
    url: 'https://message.bilibili.com/#/whisper',
    unreadCount: 0,
    icon: 'i-solar:chat-round-bold-duotone',
  },
])

onMounted(() => {
  getUnreadMessageCount()
})

function getUnreadMessageCount() {
  api.notification.getUnreadMsg().then((res) => {
    if (res.code === 0) {
      const resData = res.data

      list.value[0].unreadCount = resData.recv_reply
      list.value[1].unreadCount = resData.at
      list.value[2].unreadCount = resData.recv_like
      list.value[3].unreadCount = resData.sys_msg
    }
  }).catch(() => {
    list.value[0].unreadCount = 0
    list.value[1].unreadCount = 0
    list.value[2].unreadCount = 0
    list.value[3].unreadCount = 0
  })

  api.notification.getUnreadDm().then((res) => {
    if (res.code === 0) {
      const resData = res.data
      list.value[4].unreadCount = resData.follow_unread
    }
  }).catch(() => {
    list.value[4].unreadCount = 0
  })
}

function handleClick(event: MouseEvent, item: { name: string, url: string, unreadCount: number, icon: string }) {
  if (settings.value.openNotificationsPageAsDrawer) {
    emit('itemClick', item)
  }
}
</script>

<template>
  <div
    style="backdrop-filter: var(--bew-filter-glass-1);"
    bg="$bew-elevated"
    p="4"
    rounded="$bew-radius"
    shadow="[var(--bew-shadow-edge-glow-1),var(--bew-shadow-3)]"
    border="1 $bew-border-color"
    flex="~ col"
  >
    <ALink
      v-for="item in list"
      :key="item.name"
      :href="item.url"
      type="topBar"
      pos="relative"
      flex="~ items-center justify-between gap-2"
      p="x-4 y-2"
      bg="hover:$bew-fill-2"
      rounded="$bew-radius"
      transition="all duration-300"
      m="b-1 last:b-0"
      :custom-click-event="!settings.useOldTopBar && settings.openNotificationsPageAsDrawer"
      @click="(event: MouseEvent) => handleClick(event, item)"
    >
      <div flex="~ items-center gap-2">
        <i :class="item.icon" text="$bew-text-2" />
        <span flex="1 shrink-0" text-nowrap>{{ item.name }}</span>
      </div>
      <!-- Use visibility to control the number of notifications to prevent width changes as soon as there is a number -->
      <div
        :style="{ visibility: item.unreadCount > 0 ? 'visible' : 'hidden' }"
        bg="$bew-theme-color"
        rounded="$bew-radius"
        text="white xs leading-none center"
        grid="~ place-items-center"
        px-1
        min-w="16px"
        h="16px"
      >
        {{ item.unreadCount > 99 ? '99+' : item.unreadCount }}
      </div>
    </ALink>
  </div>
</template>
