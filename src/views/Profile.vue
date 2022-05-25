<template>
  <section class="p-4 bg-secondary w-full text-primary">
    <h1>Welcome, {{ user?.displayName }}</h1>
    <div class="mt-8" v-if="phone">
    <p>Your Default verified Mobile No. is: {{phone.phone_no}} <button @click="changePhone">&nbsp;<font-awesome-icon :icon="['fas', 'pen']" class="mr-1" /></button></p>
    </div>
    <div class="mb-4 flex" v-if="editPhone">
      <div class="md:basis-1/2">
        <label class="block text-sm font-bold mb-2 pt-4" for="newphone">
          <font-awesome-icon :icon="['fas', 'phone']" class="mr-1" />
          <span v-if="phone">Enter New Mobile No.</span><span v-else>Please add a Default Mobile No.</span>
        </label>
        <input
          class="t-input w-full pr-4"
          type="number"
          id="phone"
          v-model="newphone"
          placeholder="phone"
        />
      </div>
      <div class="md:basis-1/4" v-if="showOtpInput">
        <label class="block text-sm font-bold mb-2 pt-4" for="otp">
          Please enter otp.
        </label>
        <input
          class="t-input w-full"
          type="number"
          id="otp"
          v-model="otp"
          placeholder="otp"
        />
      </div>
      <div class="md:basis-1/4 pt-8" v-if="showOtpInput">
        <button @click="savePhone" class="text-white bg-blue-700 m-4 t-btn inline-flex items-center">Save</button>
      </div>
      <div class="md:basis-1/4 pt-8" v-if="!showOtpInput">
        <button @click="sendOtp" class="text-white bg-blue-700 m-4 t-btn inline-flex items-center">Send OTP</button>
      </div>
    </div>
    <button @click="getPincode" class="text-white bg-blue-700 m-4 t-btn inline-flex items-center">Get Address</button>
    <div>
    <pre class="text-black">
    Address: {{address}}
    </pre>
    <pre class="text-black">
    Pincode: {{pin}}
    </pre>
    </div>
  </section>
</template>

<script setup>
import { useAuthState, useSignOut } from "@/firebase";
import { useRouter } from "vue-router";
import { ref, onUpdated } from "vue";
import { Geolocation } from '@capacitor/geolocation';
import axios from 'axios';
import Loading from "@/components/Loading.vue";
const { user } = useAuthState();
const router = useRouter();
const loading = ref(false);
const address = ref('')
const pin = ref()
const phone = ref()
const newphone = ref()
const otp = ref()
const otpv = ref()
const showOtpInput = ref(false)
const editPhone = ref(false)

const sendOtp = async() => {
  if(newphone.value.toString().length === 10){
    //generates six digit random nnumber
    otpv.value = Math.floor(100000 + Math.random() * 900000);

    let res = await axios.get('https://www.fast2sms.com/dev/bulkV2?authorization=iE1fx6MVL0wYpWUto4sBuQk2bJdqjFK8Oy7GmDaTPrhAN93g5HWF3JCkDsAlOHRipSbwZqycTjMueUB0&&variables_values='+otpv.value+'&route=otp&numbers='+newphone.value)
    console.log(res)
    showOtpInput.value = true
  }
  else {
    alert('please enter valid mobile no.!')
    newphone.value = ''
  }
}

const savePhone = async() => {
  if(otp.value === otpv.value){
    let data = {
      uid: user.value.uid,
      name: user.value.displayName,
      phone_no: newphone.value.toString(),
      is_verified: 1
    }

    let res = (phone.value && phone.value.id) ? await axios.put('https://droplet.netserve.in/pws-phones/'+phone.value.id, data).then(r => r.data) : await axios.post('https://droplet.netserve.in/pws-phones', data).then(r => r.data)
    if(res) {
      editPhone.value = false
      otp.value = ''
      phone.value = res
      newphone.value = ''
      showOtpInput.value = false
    }
  }
  else{ 
    alert('Please enter correct OTP')
  }
}

const changePhone = () => {
  editPhone.value = (!editPhone.value) ? true : false
}

const signOutUser = async () => {
  loading.value = true;
  await useSignOut();
  await router.replace({ name: "Login" });
  loading.value = false;
};
// AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw
const getPincode = async() => {
  let coordinates = await Geolocation.getCurrentPosition();
  let add = await axios.get('https://maps.googleapis.com/maps/api/geocode/json?latlng='+coordinates.coords.latitude+','+coordinates.coords.longitude+'&result_type=route&key=AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw').then(r => r.data.results[0])
  address.value = add.formatted_address
  console.log(address.value)
  let data = add.address_components.filter(a => a.types.includes("postal_code"))

  pin.value = data[0].short_name
  //let pin = geocodeToPincode({lat: coordinates.coords.latitude, lng: coordinates.coords.longitude, key: 'AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw'})
  //console.log(pin)
}
onUpdated(async() => {
  if(!phone.value){
    let data = await axios.get('https://droplet.netserve.in/pws-phone?uid='+user.value.uid).then(r => r.data)
    if(data && data[0]) phone.value = data[0]
    else editPhone.value = true
  }
})
</script>