<template>
  <div class="container page d-flex justify-content-center align-items-center">
    <div>
      <div class="login-page__header">{{ $t("login.title") }}</div>
      <div class="tabs">
        <div
          class="tabs__tab-header"
          :class="{ 'active-tab': activeTab == 'individual' }"
          @click="activeTab = 'individual'"
        >
          {{ $t("login.individual") }}
        </div>
        <div
          class="tabs__tab-header"
          :class="{ 'active-tab': activeTab == 'entity' }"
          @click="activeTab = 'entity'"
        >
          {{ $t("login.entity") }}
        </div>
      </div>
      <div
        v-if="errors"
        class="
          errors-block
          d-flex
          justify-content-center
          flex-wrap
          err-warn-text
        "
      >
        {{ errors }}
      </div>
      <div v-if="activeTab == 'entity'">
        <label class="gray-label">{{ $t("login.bin") }}</label>
        <input
          type="tel"
          class="form-input login-input"
          v-model="bin"
          v-mask="'############'"
        />
      </div>
      <label class="gray-label">{{ $t("login.iin") }}</label>
      <input
        type="tel"
        class="form-input login-input"
        v-model="iin"
        v-mask="'############'"
      />
      <label class="gray-label d-flex justify-content-between flex-wrap">
        {{ $t("login.password") }}
        <nuxt-link to="/base/auth/forgotPass" class="forgotPass-link">
          {{ $t("login.forgotPassword") }}
        </nuxt-link>
      </label>
      <input
        type="password"
        class="form-input login-input"
        v-model="password"
        v-on:keyup.enter="send"
      />
      <div class="login-buttons">
        <button @click="send" class="violet-btn">
          {{ $t("login.loginBtn") }}
        </button>
        <p>{{ $t("login.or") }}</p>
        <button @click="edsSend" class="brown-btn">
          {{ $t("login.loginEcpBtn") }}
        </button>
        <p>
          {{ $t("login.loginEcpText") }}
        </p>
      </div>
      <div class="link-to-signup">
        {{ $t("login.hasntAccount") }}
        <nuxt-link to="/base/auth/register">
          {{ $t("login.signInLink") }}
        </nuxt-link>
      </div>
      <div class="link-to-arm text-center">
        <nuxt-link to="/arm/login">{{ $t("login.toArmLogin") }}</nuxt-link>
      </div>
    </div>
  </div>
</template>
<script>
import { mapActions } from "vuex";
import { mask } from "vue-the-mask";
export default {
  layout: "base",
  data() {
    return {
      iin: null,
      bin: null,
      password: null,
      activeTab: "individual",
      xml: "<authorization></authorization>",
      sign: null,
      errors: null,
    };
  },
  directives: { mask },
  methods: {
    ...mapActions("eds", ["signCMS", "signXml"]),
    async send() {
      let data = {
        iin: this.iin,
        password: this.password,
      };
      if (this.activeTab == "entity") {
        data.bin = this.bin;
      }
      await this.$auth
        .loginWith(this.activeTab, {
          data,
        })
        .then((resp) => {
          if (resp?.status == 200) {
            this.$router.push(this.$route.query?.redirect || "/base/");
          }
        })
        .catch((error) => {
          this.errors = error.response.data.message;
        });
    },
    edsSend() {
      this.signXml(this.xml).then((sign) => {
        this.sign = sign;
        this.$auth
          .loginWith(`${this.activeTab}Eds`, {
            data: {
              sign: this.sign,
            },
          })
          .then((resp) => {
            if (resp == 1) {
              this.$router.push(this.$route.query?.redirect || "/base/");
            }
          })
          .catch((error) => {
            this.errors = error.response.data.message;
          });
      });
    },
  },
};
</script>
