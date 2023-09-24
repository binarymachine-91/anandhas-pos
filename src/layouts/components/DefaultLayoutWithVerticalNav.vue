<script setup>
import { useTheme } from 'vuetify'
import VerticalNavSectionTitle from '@/@layouts/components/VerticalNavSectionTitle.vue'
import upgradeBannerDark from '@images/pro/upgrade-banner-dark.png'
import upgradeBannerLight from '@images/pro/upgrade-banner-light.png'
import VerticalNavLayout from '@layouts/components/VerticalNavLayout.vue'
import VerticalNavLink from '@layouts/components/VerticalNavLink.vue'

// Components
import Footer from '@/layouts/components/Footer.vue'
import NavbarThemeSwitcher from '@/layouts/components/NavbarThemeSwitcher.vue'
import UserProfile from '@/layouts/components/UserProfile.vue'

const vuetifyTheme = useTheme()

const upgradeBanner = computed(() => {
  return vuetifyTheme.global.name.value === 'light' ? upgradeBannerLight : upgradeBannerDark
})
</script>

<template>
  <VerticalNavLayout>
    <!-- 👉 navbar -->
    <template #navbar="{ toggleVerticalOverlayNavActive }">
      <div class="d-flex h-100 align-center">
        <!-- 👉 Vertical nav toggle in overlay mode -->
        <IconBtn
          class="ms-n3 d-lg-none"
          @click="toggleVerticalOverlayNavActive(true)"
        >
          <VIcon icon="bx-menu" />
        </IconBtn>

        <!-- 👉 Search -->
        <div
          class="d-flex align-center cursor-pointer"
          style="user-select: none;"
          v-if="false"
        >
          <!-- 👉 Search Trigger button -->
          <IconBtn>
            <VIcon icon="bx-search" />
          </IconBtn>

          <span class="d-none d-md-flex align-center text-disabled">
            <span class="me-3">Search</span>
            <span class="meta-key">&#8984;K</span>
          </span>
        </div>

        <VSpacer />

        <IconBtn
          class="me-2"
          href="https://github.com/themeselection/sneat-vuetify-vuejs-admin-template-free"
          target="_blank"
          rel="noopener noreferrer"
          v-if="false"
        >
          <VIcon icon="bxl-github" />
        </IconBtn>

        <IconBtn class="me-2" v-if="false">
          <VIcon icon="bx-bell" />
        </IconBtn>

        <NavbarThemeSwitcher v-if="false" class="me-2" />

        <UserProfile />
      </div>
    </template>

    <template #vertical-nav-content>
      <!-- Menu -->
      <VerticalNavSectionTitle
        :item="{
          heading: 'POS',
        }"
      />
      <VerticalNavLink
        :item="{
          title: 'New Bill',
          icon: 'mdi-cloud-print-outline',
          to: '/newbill',
        }"
      />
      <VerticalNavLink
        :item="{
          title: 'View Bill',
          icon: 'mdi-view-dashboard-edit-outline',
          to: '/viewbill',
        }"
      />

      <!-- Menu -->
      <VerticalNavSectionTitle
        :item="{
          heading: 'Menu',
        }"
      />
      <VerticalNavLink
        :item="{
          title: 'Menu',
          icon: 'mdi-bowl-mix',
          to: '/menu',
        }"
      />
      <VerticalNavLink
        :item="{
          title: 'Menu Item',
          icon: 'mdi-view-list-outline',
          to: '/menuitem',
        }"
      />

      <!-- User -->
      <VerticalNavSectionTitle
        :item="{
          heading: 'User Management',
        }"
      />
      <VerticalNavLink
        :item="{
          title: 'Department',
          icon: 'mdi-account-cowboy-hat-outline',
          to: '/department',
        }"
        v-if="false"
      />
      <VerticalNavLink
        :item="{
          title: 'Users',
          icon: 'mdi-account',
          to: '/users',
        }"
      />




    </template>


    <!-- 👉 Pages -->
    <slot />


  </VerticalNavLayout>
</template>

<style lang="scss" scoped>
.meta-key {
  border: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 6px;
  block-size: 1.5625rem;
  line-height: 1.3125rem;
  padding-block: 0.125rem;
  padding-inline: 0.25rem;
}
</style>
