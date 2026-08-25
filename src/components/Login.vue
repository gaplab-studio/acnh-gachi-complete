<template>
  <div class="login">
    <div class="login-header">
      <h2 class="login-title">ローカル設定・インポート</h2>
      <div class="close"><CloseButton @click="close" /></div>
    </div>
    <div class="login-body">
      <p class="local-note">取得状況はこの端末のブラウザ内だけに保存されます。ログインやクラウド同期は行いません。</p>
      <LoginCollected ref="loginCollected" />
      <LoginSettings :settings="settings" @change="changeSettings" />
    </div>
  </div>
</template>

<script>
import CloseButton from "../components/CloseButton";
import LoginCollected from "../components/LoginCollected";
import LoginSettings from "../components/LoginSettings";

export default {
  name: "Login",
  components: { CloseButton, LoginCollected, LoginSettings },
  computed: {
    settings() { return this.$store.getters.settings; },
  },
  methods: {
    close() {
      this.$refs.loginCollected.abortUpdateCollected();
      this.$emit("close");
    },
    changeSettings(newValue) { this.$store.commit("changeSettings", newValue); },
  },
};
</script>

<style lang="scss" scoped>
.login { position: fixed; z-index: 1100; top: 0; left: 0; width: 100%; height: 100%; background-color: var(--app-body-bg); overflow-y: auto; user-select: text; }
.login-header { position: relative; text-align: center; padding: 0.5rem; line-height: 64px; }
.login-title { margin-bottom: 0; }
.close { position: absolute; top: 8px; right: 16px; }
.login-body { padding: 1.5rem; max-width: 600px; margin-right: auto; margin-left: auto; text-align: center; }
.local-note { font-size: 13px; font-weight: 700; }
</style>
