<script lang="ts" setup>
const { user, signOut, appInfo } = useGlobal()
// So watcher in users store is triggered
useUsers()

const { isFeatureEnabled } = useBetaFeatureToggle()

const { leftSidebarState } = storeToRefs(useSidebarStore())

const auditsStore = useAuditsStore()

const name = computed(() => user.value?.display_name?.trim())

const isMenuOpen = ref(false)

const isAuthTokenCopied = ref(false)

const isLoggingOut = ref(false)

const { isMobileMode } = useGlobal()

const { isUIAllowed } = useRoles()

const logout = async () => {
  isLoggingOut.value = true
  try {
    const isSsoUser = !!(user?.value as any)?.sso_client_id

    await signOut({
      redirectToSignin: true,
      signinUrl: isSsoUser ? '/sso' : '/signin',
    })
  } catch (e) {
    console.error(e)
  } finally {
    isLoggingOut.value = false
  }
}

watch(isMenuOpen, () => {
  if (isAuthTokenCopied.value) {
    isAuthTokenCopied.value = false
  }
})

watch(leftSidebarState, () => {
  if (leftSidebarState.value === 'peekCloseEnd') {
    isMenuOpen.value = false
  }
})

// This is a hack to prevent github button error (prevents navigateTo if user is not signed in)
const isMounted = ref(false)

onMounted(() => {
  isMounted.value = true
})

const isExperimentalFeatureModalOpen = ref(false)

const openExperimentationMenu = () => {
  isMenuOpen.value = false
  isExperimentalFeatureModalOpen.value = true
}

const accountUrl = computed(() => {
  return isUIAllowed('superAdminSetup') && !isEeUI ? '/account/setup' : '/account/profile'
})
</script>

<template>
  <div class="flex w-full flex-col border-gray-200 gap-y-1">
    <LazyGeneralMaintenanceAlert />
    <div class="flex items-center justify-between">
      <NcDropdown v-model:visible="isMenuOpen" placement="topLeft" overlay-class-name="!min-w-64">
        <div
          class="flex flex-row py-1 px-3 gap-x-2 items-center text-gray-700 hover:bg-gray-200 rounded-lg cursor-pointer h-8"
          data-testid="nc-sidebar-userinfo"
        >
          <GeneralUserIcon :user="user" size="medium" />
          <NcTooltip class="max-w-32 truncate" show-on-truncate-only>
            <template #title>
              {{ name ? name : user?.email }}
            </template>

            {{ name ? name : user?.email }}
          </NcTooltip>

          <GeneralIcon icon="chevronDown" class="flex-none !min-w-5 transform rotate-180 !text-gray-500" />
        </div>
        <template #overlay>
          <NcMenu data-testid="nc-sidebar-userinfo" variant="small">
            <NcMenuItem data-testid="nc-sidebar-user-logout" @click="logout">
              <div v-e="['c:user:logout']" class="flex gap-2 items-center">
                <GeneralLoader v-if="isLoggingOut" class="!ml-0.5 !mr-0.5 !max-h-4.5 !-mt-0.5" />
                <GeneralIcon v-else icon="signout" class="menu-icon" />
                <span class="menu-btn"> {{ $t('general.logout') }}</span>
              </div>
            </NcMenuItem>
          </NcMenu>
        </template>
      </NcDropdown>
      <DashboardFeatureExperimentation v-model:value="isExperimentalFeatureModalOpen" />
    </div>

    <template v-if="isMobileMode || appInfo.ee"></template>
    <div v-else class="flex flex-row w-full justify-between pt-0.5 truncate">
    </div>
  </div>
</template>

<style lang="scss" scoped>
.menu-btn {
  line-height: 1.5;
}
.menu-icon {
  @apply w-4 h-4;
  font-size: 1rem;
}

.social-icon {
  @apply my-0.5 w-4 h-4 stroke-transparent;
  // Make icon black and white
  filter: grayscale(100%);

  // Make icon red on hover
  &:hover {
    filter: grayscale(100%) invert(100%);
  }
}

.social-icon-wrapper {
  .nc-icon {
    @apply mr-0.15;
  }

  &:hover {
    .social-icon {
      filter: none !important;
    }
  }
}
</style>

<style lang="scss">
.nc-lang-menu-overlay {
  .ant-popover-inner {
    @apply !rounded-lg;
  }

  .ant-popover-inner-content {
    @apply !bg-transparent;
  }
}
</style>
