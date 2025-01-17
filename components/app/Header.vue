<template>
  <div>
    <header
      class="flex space-x-2 flex-1 py-2 px-2 items-center justify-between"
    >
      <div class="space-x-2 inline-flex items-center">
        <ButtonSecondary
          class="tracking-wide !font-bold !text-secondaryDark"
          label="HOPPSCOTCH"
          to="/"
        />
        <AppGitHubStarButton class="mt-1.5 transition" />
      </div>
      <div class="space-x-2 inline-flex items-center">
        <ButtonSecondary
          id="installPWA"
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('header.install_pwa')"
          svg="download"
          class="rounded"
          @click.native="showInstallPrompt()"
        />
        <ButtonSecondary
          v-if="currentUser === null"
          svg="upload-cloud"
          :label="$t('header.save_workspace')"
          filled
          class="hidden !font-semibold md:flex"
          @click.native="showLogin = true"
        />
        <ButtonPrimary
          v-if="currentUser === null"
          :label="$t('header.login')"
          @click.native="showLogin = true"
        />
        <span v-else class="pr-2">
          <tippy ref="user" interactive trigger="click" theme="popover" arrow>
            <template #trigger>
              <ProfilePicture
                v-if="currentUser.photoURL"
                v-tippy="{
                  theme: 'tooltip',
                }"
                :url="currentUser.photoURL"
                :alt="currentUser.displayName"
                :title="currentUser.displayName"
                indicator
                :indicator-styles="isOnLine ? 'bg-green-500' : 'bg-red-500'"
              />
              <ButtonSecondary
                v-else
                v-tippy="{ theme: 'tooltip' }"
                :title="$t('header.account')"
                class="rounded"
                svg="user"
              />
            </template>
            <SmartItem
              to="/settings"
              svg="settings"
              :label="$t('navigation.settings')"
              @click.native="$refs.user.tippy().hide()"
            />
            <FirebaseLogout @confirm-logout="$refs.user.tippy().hide()" />
          </tippy>
        </span>
      </div>
    </header>
    <AppAnnouncement v-if="!isOnLine" />
    <FirebaseLogin :show="showLogin" @hide-modal="showLogin = false" />
  </div>
</template>

<script>
import { defineComponent } from "@nuxtjs/composition-api"
import intializePwa from "~/helpers/pwa"
import { currentUser$ } from "~/helpers/fb/auth"
import { getLocalConfig, setLocalConfig } from "~/newstore/localpersistence"
import { useReadonlyStream } from "~/helpers/utils/composables"

export default defineComponent({
  setup() {
    return {
      currentUser: useReadonlyStream(currentUser$, null),
    }
  },
  data() {
    return {
      // Once the PWA code is initialized, this holds a method
      // that can be called to show the user the installation
      // prompt.
      showInstallPrompt: null,
      showLogin: false,
      isOnLine: navigator.onLine,
    }
  },
  async mounted() {
    window.addEventListener("online", () => {
      this.isOnLine = true
    })
    window.addEventListener("offline", () => {
      this.isOnLine = false
    })

    // Initializes the PWA code - checks if the app is installed,
    // etc.
    this.showInstallPrompt = await intializePwa()

    const cookiesAllowed = getLocalConfig("cookiesAllowed") === "yes"
    if (!cookiesAllowed) {
      this.$toast.show(this.$t("app.we_use_cookies").toString(), {
        icon: "cookie",
        duration: 0,
        action: [
          {
            text: this.$t("action.learn_more").toString(),
            onClick: (_, toastObject) => {
              setLocalConfig("cookiesAllowed", "yes")
              toastObject.goAway(0)
              window
                .open("https://docs.hoppscotch.io/privacy", "_blank")
                .focus()
            },
          },
          {
            text: this.$t("action.dismiss").toString(),
            onClick: (_, toastObject) => {
              setLocalConfig("cookiesAllowed", "yes")
              toastObject.goAway(0)
            },
          },
        ],
      })
    }
  },
})
</script>
