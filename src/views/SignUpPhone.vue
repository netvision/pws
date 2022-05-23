<template>
<section class="t-auth-section">
    <h1 class="mb-6 text-primary text-center">Sign Up</h1>
    <hr />
    <div class="mb-4">
        <label class="block text-sm font-bold mb-2 text-primary" for="phone">
          <font-awesome-icon :icon="['fas', 'phone']" class="mr-1" />
          Mobile No
        </label>
        <input
          :class="`t-input w-full ${!validNo && 'border border-error'}`"
          type="number"
          id="phone"
          placeholder="phone"
          v-model="phone"
          required
        />
        <p class="mt-0.5 text-error text-xs italic" v-show="!validNo">
          Enter Valid No
        </p>
        <button class="text-primary" @click="sendOtp" id="sendOtp">Send OTP</button>
      </div>
      <div>
      <label class="block text-sm font-bold mb-2 text-primary" for="otp">
          
          OTP
        </label>
        <input
          :class="`t-input w-full ${!validNo && 'border border-error'}`"
          type="number"
          id="otp"
          placeholder="OTP"
          v-model="otp"
          required
        />
        <p class="mt-0.5 text-error text-xs italic" v-show="!validNo">
          Enter OTP
        </p>
        <button class="text-primary" @click="verifyCode" id="verify">Verify OTP</button>
      </div>
      <div id="sign-in-button"></div>
</section>
</template>
<script setup>
import { getAuth, RecaptchaVerifier, signInWithPhoneNumber } from "firebase/auth";

import {ref, onMounted } from 'vue'

const validNo = ref(true)
const phone = ref()
const otp = ref()
const auth = getAuth()
const appVerifier = ref()
const sendOtp = () => {
    let ph = '+91'+phone.value
    appVerifier.value = window.recaptchaVerifier
    signInWithPhoneNumber(auth, ph, appVerifier.value)
    .then((confirmationResult) => {
        console.log(confirmationResult)
      // SMS sent. Prompt user to type the code from the message, then sign the
      // user in with confirmationResult.confirm(code).
      window.confirmationResult = confirmationResult;
      // ...
    }).catch((error) => {
        console.log(error)
      // Error; SMS not sent
      // ...
    });
}

const verifyCode = () => {
    window.confirmationResult.confirm(otp.value).then((result) => {
  // User signed in successfully.
  const user = result.user;
  console.log(user)
  // ...
}).catch((error) => {
  // User couldn't sign in (bad verification code?)
  // ...
});
}

onMounted(() => {
    window.recaptchaVerifier = new RecaptchaVerifier('sign-in-button', {
        'size': 'invisible',
        'callback': (response) => {
            ///
        }
        }, auth);
})
</script>