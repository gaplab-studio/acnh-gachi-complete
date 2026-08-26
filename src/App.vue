<template>
  <div id="app" :class="{ isSearchMode: isSearchMode }">
    <transition name="slide">
      <Drawer :navs="navs" :active="activeNav" v-show="isOpenDrawer" />
    </transition>
    <transition name="fade">
      <span class="backdrop" v-show="isOpenDrawer" @click="toggleDrawer" />
    </transition>
    <div class="nav" v-show="!isSearchMode">
      <Button @click="toggleDrawer">
        <img v-show="!isOpenDrawer" src="./assets/menu.svg" />
        <svg
          v-show="isOpenDrawer"
          xmlns="http://www.w3.org/2000/svg"
          width="28"
          height="28"
          viewBox="0 0 50 50"
        >
          <path
            fill="#42b983"
            d="M9.016 40.837a1.001 1.001 0 001.415-.001l14.292-14.309 14.292 14.309a1 1 0 001.416-1.413L26.153 25.129 40.43 10.836a1 1 0 10-1.415-1.413L24.722 23.732 10.43 9.423a1 1 0 10-1.415 1.413l14.276 14.293L9.015 39.423a1 1 0 00.001 1.414z"
          />
        </svg>
        <span class="name"><span>あつ森</span>ガチコンプ</span>
      </Button>
    </div>
    <div v-if="g001ImportSummary" class="g001-import-summary" role="status">
      {{ g001ImportSummary }}
    </div>
    <router-view v-if="isLoadComplete" />
  </div>
</template>
<script>
import { navs } from "./utils/navs";
import itemsJson from "./assets/items.json";
import { syncData, loadFirebaseData } from "./utils/db.js";
import Drawer from "./components/Drawer.vue";
import Button from "./components/Button.vue";

export default {
  components: {
    Drawer,
    Button,
  },
  data() {
    return {
      isLoadComplete: null,
      g001ImportSummary: "",
      navs,
    };
  },
  computed: {
    activeNav() {
      return this.$store.getters.activeNav;
    },
    isOpenDrawer() {
      return this.$store.getters.isOpenDrawer;
    },
    isLogin() {
      return this.$store.getters.isLogin;
    },
    localCollected() {
      return this.$store.getters.localCollectedData;
    },
    cloudUpdateIndex() {
      return this.$store.getters.cloudUpdateIndex;
    },
    isDoneSyncCloudFirstTime() {
      return this.$store.getters.isDoneSyncCloudFirstTime;
    },
    isSearchMode() {
      return this.$store.getters.isSearchMode;
    },
  },
  watch: {
    isLogin() {
      loadFirebaseData();
    },
    cloudUpdateIndex() {
      if (!this.isDoneSyncCloudFirstTime) {
        syncData();
        this.$store.commit("isDoneSyncCloudFirstTime", true);
      }
    },
  },
  async created() {
    await Promise.all([this.loadLocalStorageData()]);
    this.importG001Fragment();
    this.isLoadComplete = true;
  },
  methods: {
    toggleDrawer() {
      this.$store.commit("toggleDrawer");
    },
    async loadLocalStorageData() {
      const self = this;
      let [collected, updateIndex, wishlist, stocklist, settings, partnerlist] =
        await Promise.all([
          self.$vlf.getItem("collected"),
          self.$vlf.getItem("updateIndex"),
          self.$vlf.getItem("wishlist"),
          self.$vlf.getItem("stocklist"),
          self.$vlf.getItem("settings"),
          self.$vlf.getItem("partnerlist"),
        ]);
      collected = collected || {};
      updateIndex = updateIndex || 0;
      self.$store.commit(
        "changeSettings",
        settings || {
          isFullMode: false,
        }
      );
      self.$store.commit("initLocalCollectedData", { collected, updateIndex });
      self.$store.commit("initWishlist", wishlist);
      self.$store.commit("initStocklist", stocklist);
      self.$store.commit("initPartnerlist", partnerlist);
      return self.localCollected;
    },
    importG001Fragment() {
      const match = window.location.hash.match(/^#g001=([A-Za-z0-9_-]+)$/);
      const encodedPayload = window.__G001_IMPORT_FRAGMENT__ || (match && match[1]);
      if (!encodedPayload) return;
      try {
        const encoded = encodedPayload.replace(/-/g, "+").replace(/_/g, "/");
        const json = atob(encoded + "=".repeat((4 - (encoded.length % 4)) % 4));
        const payload = JSON.parse(json);
        const collected = this.$store.getters.localCollectedData || {};
        const updates = {};
        let matched = 0;
        let eligible = 0;
        Object.entries(payload).forEach(([name, value]) => {
          const item = itemsJson.find((candidate) => candidate.name === name);
          if (item) matched += 1;
          const itemLength = item && item.variants ? item.variants.length : 1;
          if (value === "0" && item && itemLength === 1) {
            eligible += 1;
            const key = item.uniqueEntryId || item.name;
            if (!collected[key]) updates[key] = "0";
          }
        });
        if (Object.keys(updates).length) {
          this.$store.commit("updateLocalCollectedDataBatch", {
            items: Object.keys(updates),
            collectedArray: Object.values(updates),
          });
        }
        this.g001ImportSummary = `CatalogScannerから${Object.keys(updates).length}件を取得済みに追加しました。（照合${matched}件・反映対象${eligible}件）`;
      } catch (error) {
        this.g001ImportSummary = "CatalogScannerの反映データを読み込めませんでした。もう一度「ガチコンプへ反映」を実行してください。";
      } finally {
        window.history.replaceState(null, "", window.location.pathname);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
#app {
  padding-top: 52px;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  &.isSearchMode {
    padding-top: 0;
  }
}

.nav {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  z-index: 1010;
  padding: 4px 8px;
  user-select: none;
  background-color: var(--app-body-bg);
  font-size: 15px;
  font-weight: bold;
}

.name {
  margin-left: 4px;
  color: #42b983;

  span {
    @media (max-width: 359.98px) {
      display: none;
    }
  }
}

.g001-import-summary {
  position: fixed;
  top: 60px;
  right: 12px;
  left: 12px;
  z-index: 1011;
  padding: 10px 12px;
  border: 1px solid #42b983;
  border-radius: 8px;
  background: #f0fff8;
  color: #17663f;
  font-size: 14px;
}

.backdrop {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1009;
  background-color: rgba(0, 0, 0, 0.5);
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.15s;
}
.slide-enter,
.slide-leave-to {
  transform: translateY(-100%);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.15s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
