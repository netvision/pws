<template>
  <section class="p-4 bg-secondary w-full">
    <p class="mt-4 text-center">
      
    </p>
    <p class="mt-4 text-center">
      <img src="@/assets/water_tanker_logo.jpg" /><br />
      <router-link to="/book" class="t-btn bg-primary">Book a Tanker</router-link>
    </p>
    <p class="text-center pt-2"><span v-if="!user" class="text-primary"><a href="/auth/login">Sign In</a> |  <a href="/auth/signup">Sign Up</a></span></p>
  </section>
</template>
<script setup>
import { useAuthState } from "@/firebase";

const { user } = useAuthState();

</script>
<style lang="postcss" scoped>
code {
  @apply bg-gray-800 px-2 mx-2;
}
</style>

