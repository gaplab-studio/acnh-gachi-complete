<template>
  <div class="login">
    <div class="login-header">
      <h2 class="login-title">
        <template v-if="isLogin">設定</template>
        <template v-else>ログイン</template>
      </h2>
      <div class="close"><CloseButton @click="close" /></div>
    </div>
    <div class="login-body">
      <template v-if="isLogin && user">
        <img :src="user.photoURL" alt="Avatar" class="avatar" />
      </template>
      <div v-else-if="isLogin === false" class="mb-6">
        <p style="font-size: 14px">
          ログインすると、この端末の取得状況をクラウドへ同期し、公開・X共有・他ユーザーの公開状況との比較を利用できます。
        </p>
        <Button block @click="login('google')">
          <img src="../assets/google.svg" width="24px" height="24px" style="margin-right: 1rem" />
          Google でログイン
        </Button>
        <p style="font-size: 12px; margin-block: 2rem">
          X（旧Twitter）ログインは提供元APIの変更により利用できません。Googleログインを使用してください。
        </p>
      </div>

      <LoginCollected ref="loginCollected" />
      <LoginSettings :settings="settings" @change="changeSettings" />

      <template v-if="isLogin && user">
        <LoginName :userName="userName" @change="saveName" />
        <LoginIslandName :islandName="islandName" @change="saveIslandName" />
        <LoginShare />
        <Card title="ID">
          <p>{{ user.uid }}</p>
          <p class="small">公開URLと比較機能で使う固有IDです。</p>
        </Card>
        <Button block @click="logout">ログアウト</Button>
        <p class="small mt-4">ログアウトするとこのブラウザのローカルデータを消去します。クラウドのデータはGoogleログインで復元できます。</p>
        <LoginDeleteAccount :userName="userName" :user="user" :logout="logout" />
      </template>
    </div>
  </div>
</template>

<script>
import firebase from "../plugins/firebase";
import Auth from "../utils/auth";
import { syncData } from "../utils/db.js";
import CloseButton from "../components/CloseButton";
import Button from "../components/Button";
import Card from "../components/Card";
import LoginName from "../components/LoginName";
import LoginIslandName from "../components/LoginIslandName";
import LoginShare from "../components/LoginShare";
import LoginCollected from "../components/LoginCollected";
import LoginSettings from "../components/LoginSettings";
import LoginDeleteAccount from "../components/LoginDeleteAccount";

const db = firebase.firestore();

export default {
  name: "Login",
  components: {
    CloseButton, Button, Card, LoginName, LoginIslandName, LoginShare,
    LoginCollected, LoginSettings, LoginDeleteAccount,
  },
  computed: {
    user() { return this.$store.getters.user; },
    userName() { return this.$store.getters.userName; },
    islandName() { return this.$store.getters.islandName; },
    isLogin() { return this.$store.getters.isLogin; },
    settings() { return this.$store.getters.settings; },
  },
  methods: {
    login(provider) {
      Auth.login(provider, () => this.$emit("close"));
    },
    logout() {
      syncData();
      Auth.logout();
      this.$vlf.removeItems([
        "collected", "wishlist", "stocklist", "updateIndex", "islandName", "filter",
      ]).then(() => window.location.reload());
    },
    close() {
      this.$refs.loginCollected.abortUpdateCollected();
      this.$emit("close");
    },
    saveName(newName) {
      db.collection("users").doc(this.user.uid).update({ userName: newName });
      this.$store.commit("updateUserName", newName);
    },
    saveIslandName(newName) {
      db.collection("users").doc(this.user.uid).update({ islandName: newName });
      this.$store.commit("updateIslandName", newName);
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
.avatar { width: 80px; height: 80px; border-radius: 50%; margin-bottom: 2rem; }
</style>
