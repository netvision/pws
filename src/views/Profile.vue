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
    <button @click="getPincode" class="text-black">Pin</button>
    <div>
    <pre class="text-black">
    Pincode: {{pin}}
    </pre>
    
    </div>
  </section>
</template>

<script setup>
import { useAuthState, useSignOut } from "@/firebase";
import { useRouter } from "vue-router";
import { ref } from "vue";
import { Geolocation } from '@capacitor/geolocation';
import axios from 'axios';
import Loading from "@/components/Loading.vue";
const { user } = useAuthState();
const router = useRouter();

const loading = ref(false);
const address = ref('')
const pin = ref([])

const signOutUser = async () => {
  loading.value = true;
  await useSignOut();
  await router.replace({ name: "Login" });
  loading.value = false;
};
// AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw
const getPincode = async() => {
  let coordinates = await Geolocation.getCurrentPosition();
  let add = await axios.get('https://maps.googleapis.com/maps/api/geocode/json?latlng='+coordinates.coords.latitude+','+coordinates.coords.longitude+'&result_type=route&key=AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw').then(r => r.data)
  address.value = add.results[0]

  let data = address.value.address_components.filter(a => a.types.includes("postal_code"))

  pin.value = data[0].short_name
  //let pin = geocodeToPincode({lat: coordinates.coords.latitude, lng: coordinates.coords.longitude, key: 'AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw'})
  //console.log(pin)
}
</script>