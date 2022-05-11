<template>
  <div class="container page d-flex justify-content-center align-items-center">
    <div>
      <div class="login-page__header">{{ $t("register.title") }}</div>
      <div class="tabs">
        <div
          class="tabs__tab-header"
          :class="{ 'active-tab': activeTab == 'individual' }"
          @click="activeTab = 'individual'"
        >
          {{ $t("register.individual") }}
        </div>
        <div
          class="tabs__tab-header"
          :class="{ 'active-tab': activeTab == 'entity' }"
          @click="activeTab = 'entity'"
        >
          {{ $t("register.entity") }}
        </div>
      </div>
      <div
        v-if="errors"
        class="errors-block d-flex justify-content-center flex-wrap"
      >
        <template v-for="error in errors">
          <div class="err-warn-text" v-for="err in error">
            {{ err }}
          </div>
        </template>
      </div>
      <label class="gray-label d-flex justify-content-between flex-wrap">
        {{ $t("register.phone") }}
      </label>
      <input
        type="tel"
        v-mask="'+7(7##)###-##-##'"
        class="form-input login-input"
        v-model="phone"
      />
      <label class="gray-label d-flex justify-content-between flex-wrap">
        {{ $t("register.email") }}
      </label>
      <input type="email" class="form-input login-input" v-model="email" />
      <label class="gray-label d-flex flex-wrap">
        {{ $t("register.password") }}
        <span class="pass-help-ico">
          ?
          <span class="pass-help-text">
            {{ $t("register.passHelp") }}
          </span>
        </span>
      </label>
      <input
        type="password"
        class="form-input login-input"
        v-model="password"
      />
      <label class="gray-label d-flex justify-content-between flex-wrap">
        {{ $t("register.password_confirmation") }}
      </label>
      <input
        type="password"
        class="form-input login-input"
        v-model="password_confirmation"
      />
      <div class="login-buttons">
        <button @click="send" class="brown-btn">
          {{ $t("register.loginEcpBtn") }}
        </button>
      </div>
      <div class="link-to-signup">
        {{ $t("register.hasntAccount") }}
        <nuxt-link to="/base/auth/login">
          {{ $t("register.signInLink") }}
        </nuxt-link>
      </div>
    </div>
  </div>
</template>
<script>
import { mask } from "vue-the-mask";
import { mapGetters, mapActions } from "vuex";
export default {
  layout: "base",
  directives: { mask },
  data() {
    return {
      email: null,
      phone: null,
      password: null,
      password_confirmation: null,
      role: false,
      activeTab: "individual",
      xml: "<registration></registration>",
      errors: null,
    };
  },
  methods: {
    ...mapActions("eds", ["signCMS", "signXml"]),
    send() {
      let response = 0;
      if (this.activeTab == "individual") {
        this.$axios.$get("/sanctum/csrf-cookie").then((res) => {
          this.signXml(this.xml).then((sign) => {
            const params = {
              phone: this.phone,
              email: this.email,
              password: this.password,
              password_confirmation: this.password_confirmation,
              sign: sign,
            };
            this.$axios
              .$post("api/register/eds/individual", params)
              .then((r) => {
                response = this.$auth.loginWith("individual", {
                  data: {
                    iin: r.iin,
                    password: this.password,
                  },
                });
                if (response == 1) {
                  this.$router.push("/base/");
                }
              })
              .catch((error) => {
                this.errors = error.response.data.errors;
              });
          });
        });
      } else {
        this.$axios.$get("/sanctum/csrf-cookie").then((res) => {
          this.signXml(this.xml).then((sign) => {
            const params = {
              phone: this.phone,
              email: this.email,
              password: this.password,
              password_confirmation: this.password_confirmation,
              sign: sign,
            };
            this.$axios
              .$post("api/register/eds/entity", params)
              .then((r) => {
                response = this.$auth
                  .loginWith("entity", {
                    data: {
                      iin: r.iin,
                      bin: r.bin,
                      password: this.password,
                    },
                  })
                  .catch((error) => {
                    this.errors = error.response.data.errors;
                  });
                if (response == 1) {
                  this.$router.push("/base/");
                }
              })
              .catch((error) => {
                this.errors = error.response.data.errors;
              });
          });
        });
      }
    },
  },
};
</script>
