<template>
  <section class="p-4 bg-secondary w-full">
    <h1>Welcome, {{ user?.displayName }}</h1>
    <button
      class="my-4 t-btn inline-flex items-center bg-error"
      @click="signOutUser"
    >
      <Loading class="h-5 w-5" v-if="loading" />
      <font-awesome-icon :icon="['fas', 'sign-out-alt']" class="mr-2" v-else />
      LogOut
    </button>

    <div>
      <h1>Geolocation</h1>
      <p>Your location is:</p>
      <p>Latitude: {{ loc.lat }}</p>
      <p>Longitude: {{ loc.long }}</p>

      <button @click="getCurrentPosition">
        Get Current Location
      </button>
    </div>
  </section>
</template>

<script>
import { useAuthState, useSignOut } from "@/firebase";
import { useRouter } from "vue-router";
import { Geolocation } from '@capacitor/geolocation';
import { defineComponent, ref } from "vue";
import Dialog from "@/components/Dialog.vue";
import Loading from "@/components/Loading.vue";

export default defineComponent({
  name: "Home",
  components: {
    Dialog,
    Loading,
  },
  setup() {
    const { user } = useAuthState();
    const router = useRouter();

    const loading = ref(false);

    const loc = ref({
      lat: null,
      long: null,
    });

    const getCurrentPosition = async () => {
      const pos = await Geolocation.getCurrentPosition();
      loc.value = {
        lat: pos.coords.latitude,
        long: pos.coords.longitude,
      };
    };

    const signOutUser = async () => {
      loading.value = true;
      await useSignOut();
      await router.replace({ name: "Login" });
      loading.value = false;
    };

    return { user, signOutUser, loading, loc, getCurrentPosition };
  },
});
</script>

