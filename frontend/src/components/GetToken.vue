<template>
  <v-form ref="form">
    <v-container>
      <v-layout>
        <v-flex xs12 md5>
          <v-form v-model="tokenFormValid">
            <v-text-field
              v-model="serviceClient"
              label="Service Client"
              value="GETOK_SERVICE"
              readonly
              required
            ></v-text-field>

            <v-text-field
              v-model="password"
              type="password"
              label="Password"
              required
              :rules="passwordRules"
            ></v-text-field>

            <v-btn color="success" @click="handleSubmit" :disabled="!tokenFormValid">Submit</v-btn>
          </v-form>
        </v-flex>
        <v-flex xs12 md6 offset-md1>
          <v-textarea
            v-model="tokenResponse"
            rows="1"
            placeholder="This field will be filled by the token"
            auto-grow
            readonly
            label="Token Response"
            :class="jsonTextClass"
          ></v-textarea>
        </v-flex>
      </v-layout>
    </v-container>
  </v-form>
</template>

<script>
export default {
  data() {
    return {
      tokenFormValid: false,
      serviceClient: "GETOK_SERVICE",
      password: "",
      tokenResponse: "",
      jsonTextClass: "",
      passwordRules: [v => !!v || "Password is required"]
    };
  },
  methods: {
    async handleSubmit() {
      const fetchedToken = await this.fetchToken();
      this.tokenResponse = JSON.stringify(fetchedToken, null, 2);
      this.$store.commit("setToken", fetchedToken.access_token);
      this.jsonTextClass = "jsonText";
    },
    async fetchToken() {
      try {
        const url = `https://i1api.nrs.gov.bc.ca/oauth2/v1/oauth/token?disableDeveloperFilter=true&grant_type=client_credentials&scope=WEBADE-REST.*`;

        const headers = new Headers();
        headers.set(
          "Authorization",
          "Basic " + window.btoa("GETOK_SERVICE" + ":" + this.password)
        );
        const response = await fetch(url, { method: "get", headers: headers });

        const body = await response.json();

        return body;
      } catch (e) {
        console.log(`ERROR, caught error fetching from WebADE Token endpoint`); // eslint-disable-line no-console
        console.log(e); // eslint-disable-line no-console
      }
    }
  }
};
</script>

<style>
</style>
