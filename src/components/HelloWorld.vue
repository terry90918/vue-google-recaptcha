<template>
  <div>
    <h1 style="text-align: center">Vue Recaptcha Demo</h1>
    <div class="content">
      <div v-show="!sucessfulServerResponse" class="auth-form-wrapper">
        <form class="auth-form" @submit.prevent="submit">
          <div class="header">
            <h1>
              Sign up
            </h1>
          </div>
          <div class="input-group">
            <label for="email">email</label>
            <input
              v-model="email"
              required
              placeholder="name@domain.com"
              class=""
              name="email"
              id="email"
            />
          </div>
          <div class="input-group">
            <label for="password">password</label>
            <input
              required
              v-model="password"
              type="password"
              placeholder="password"
              class=""
              name="password"
              id="password"
            />
          </div>
          <div class="input-group">
            <label for="password-confirmation">confirm password</label>
            <input
              required
              type="password"
              placeholder="confirm password"
              class=""
              name="password confirmation"
              id="password-confirmation"
            />
          </div>
          <div class="input-group">
            <vue-recaptcha
              ref="recaptcha"
              @verify="onCaptchaVerified"
              @expired="onCaptchaExpired"
              size="invisible"
              sitekey="6LfJA0gUAAAAAA4YDXBubWMll55x5xLpyzu6n60G"
            >
            </vue-recaptcha>
            <button
              :disabled="status === 'submitting'"
              type="submit"
              class="button"
            >
              sign up
            </button>
          </div>
          <div v-cloak class="">
            <div v-show="serverError">
              {{ serverError }}
            </div>
          </div>
        </form>
      </div>
      <div
        v-show="sucessfulServerResponse"
        class="successful-server-response-wrapper"
      >
        <div class="successful-server-response">
          {{ sucessfulServerResponse }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import VueRecaptcha from 'vue-recaptcha'

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  components: {
    VueRecaptcha,
  },
  data() {
    return {
      email: '',
      password: '',
      passwordConfirmation: '',
      status: '',
      sucessfulServerResponse: '',
      serverError: '',
    }
  },
  methods: {
    submit() {
      // this.status = "submitting";
      this.$refs.recaptcha.execute()
    },
    onCaptchaVerified(recaptchaToken) {
      const self = this
      self.status = 'submitting'
      self.$refs.recaptcha.reset()
      axios
        .post('https://vue-recaptcha-demo.herokuapp.com/signup', {
          recaptchaToken: recaptchaToken,
        })
        .then(response => {
          self.sucessfulServerResponse = response.data.message
        })
        .catch(err => {
          self.serverError = getErrorMessage(err)

          //helper to get a displayable message to the user
          function getErrorMessage(err) {
            let responseBody
            responseBody = err.response
            if (!responseBody) {
              responseBody = err
            } else {
              responseBody = err.response.data || responseBody
            }
            return responseBody.message || JSON.stringify(responseBody)
          }
        })
        .then(() => {
          self.status = ''
        })
    },
    onCaptchaExpired() {
      this.$refs.recaptcha.reset()
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
