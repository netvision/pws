<template>
  <section class="p-4 bg-secondary w-full">
    <h1 class="text-center text-black">Book a tanker</h1>
   <div class="flex justify-around mt-5" v-if="products">
      <div class="form-check" v-for="item in products" :key="item.id">
        <input class="form-check-input appearance-none rounded-full h-4 w-4 border border-gray-300 bg-white checked:bg-blue-600 checked:border-blue-600 focus:outline-none transition duration-200 mt-1 align-top bg-no-repeat bg-center bg-contain float-left mr-2 cursor-pointer" type="radio" name="product_id" id="flexRadioDefault1" :value="item.id" v-model="order.product_id">
        <label class="form-check-label inline-block text-gray-800" for="flexRadioDefault1">
          {{item.name}} <br />
          {{item.description}}<br />
          Rs. {{item.price}}
        </label>
      </div>
      
  </div>
    <div class="mt-4 text-black">
        <label class="block text-sm font-bold mb-2 pt-4" for="name">
          <span>Full Name.:</span>
        </label>
        <input
          class="t-input w-full pr-4"
          type="text"
          id="name"
          v-model="order.name"
          placeholder="Name" 
        />
    </div>
    
    <div class="mt-4 text-black">
      <label class="block text-sm font-bold mb-2 pt-4" for="phone">
        <span>Contact No.:</span><font-awesome-icon :icon="['fas', 'phone']" class="mr-1" />
      </label>
      <input
        class="t-input w-full pr-4"
        type="text"
        id="phone"
        v-model="order.phone_no"
        placeholder="phone"
        @blur = "verifyNo" 
      />
    </div>
    <div v-if="otpInput" class="mt-4 text-black">
      <label class="block text-sm font-bold mb-2 pt-4" for="otp">
        <span>OTP:</span>
      </label>
      <input
        class="t-input w-full pr-4"
        type="number"
        id="otp"
        v-model="otpv"
        placeholder="OTP"
        @blur = "otpVerify"
      />
    </div>
    <div v-if="otpVerified" class="bg-green-100 rounded-lg py-5 px-6 mb-4 text-base text-green-700" role="alert">
      Phone no is verified!
    </div>
    <div class="mt-4 text-black form-group">
        <label class="block text-sm font-bold mb-2 pt-4" for="address">
          <span>Full Address:</span> <button @click="getPincode"> Use Current Location</button>
        </label>
        <input
          class="t-input w-full pr-4 form-control"
          type="text"
          id="address"
          v-model="order.address"
          placeholder="Address" 
        />
    </div>
    <div class="grid grid-cols-2 gap-4 mt-4">
      <div class="form-group mb-6">
        <input type="text" class="form-control
          block
          w-full
          px-3
          py-1.5
          text-base
          font-normal
          text-gray-700
          bg-white bg-clip-padding
          border border-solid border-gray-300
          rounded
          transition
          ease-in-out
          m-0
          focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none" id="exampleInput123"
          aria-describedby="town" placeholder="Town/Village"
          v-model = "order.town">
      </div>
      <div class="form-group mb-6">
        <input type="text" class="form-control
          block
          w-full
          px-3
          py-1.5
          text-base
          font-normal
          text-gray-700
          bg-white bg-clip-padding
          border border-solid border-gray-300
          rounded
          transition
          ease-in-out
          m-0
          focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none" id="exampleInput124"
          aria-describedby="pincode" placeholder="pincode"
          v-model = "order.pincode">
      </div>
    </div>
    <div>
    <button type="button" :disabled="isDisabled"
        class="mt-4 t-btn w-full inline-flex items-center bg-primary text-white" @click="saveOrder">Book</button>
    </div>
  </section>
</template>
<script setup>
import { useAuthState } from "@/firebase";
import { useRouter } from "vue-router";
import { ref, onUpdated, onMounted } from "vue";
import { Geolocation } from '@capacitor/geolocation';
import axios from 'axios';
import Loading from "@/components/Loading.vue";

const { user } = useAuthState();
const router = useRouter();
const loading = ref(false);
const products = ref()
const defaultPhone = ref();
const address = ref();
const pin = ref();
const order = ref({});
const otp = ref();
const otpv = ref();
const smsKey = import.meta.env.VITE_FAST_SMS_KEY
const otpInput = ref(false)
const otpVerified = ref(false)
const isDisabled = ref(true)

const verifyNo = async() => {
  if(order.value.phone_no.toString().length === 10){
    if(!defaultPhone.value || order.value.phone_no != defaultPhone.value.phone_no){
    
    //generates six digit random nnumber
      otp.value = Math.floor(100000 + Math.random() * 900000);

      let res = await axios.get('https://www.fast2sms.com/dev/bulkV2?authorization='+smsKey+'&variables_values='+otp.value+'&route=otp&numbers='+order.value.phone_no)
      if(res){
        alert('You will recieve OTP on this number. Please input that in the field below to verify mibile no.!')
        otpInput.value = true
      }
      
    }
  }
  else {
    alert('please enter valid mobile no.!')
    order.value.phone_no = ''
  }
}

const otpVerify = () => {
  if(otp.value == otpv.value){
    otpVerified.value = true
    otpInput.value = false
    isDisabled.value = false
  }
  else {
    alert('Please enter correct OTP or change Mobile No.')
    otpv.value = ''
  }
}

const getPincode = async() => {
  let coordinates = await Geolocation.getCurrentPosition();
  order.value.latitude = coordinates.coords.latitude;
  order.value.longitude = coordinates.coords.longitude
  let add = await axios.get('https://maps.googleapis.com/maps/api/geocode/json?latlng='+coordinates.coords.latitude+','+coordinates.coords.longitude+'&result_type=route&key=AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw').then(r => r.data.results[0])
  order.value.address = add.formatted_address
  console.log(add)
  let data = add.address_components.filter(a => a.types.includes("postal_code"))
  let townComp = add.address_components.filter(a => a.types.includes("locality"))
  
  order.value.town = townComp[0].short_name
  order.value.pincode = data[0].short_name
  //let pin = geocodeToPincode({lat: coordinates.coords.latitude, lng: coordinates.coords.longitude, key: 'AIzaSyDG9I2B0BJsIx9gtd7Hw9_9_0QA4xtBHnw'})
  //console.log(pin)
}

const saveOrder = async() => {
  let date = new Date();
  order.value.ordered_at = date.toISOString().slice(0, 19).replace('T', ' ');
  order.value.requested_at = date.toISOString().slice(0, 19).replace('T', ' ');
  order.value.user_uid = (user.value) ? user.value.uid : null;
  order.value.phone_no = order.value.phone_no.toString()
  order.value.payment_type = 'cod';
  //console.log(order.value)
  let res = await axios.post('https://droplet.netserve.in/pws-orders', order.value).then(r => r.data);
  if (res.id) {
    //send sms to the owner
    let out = await axios.get('https://www.fast2sms.com/dev/bulkV2?authorization=iE1fx6MVL0wYpWUto4sBuQk2bJdqjFK8Oy7GmDaTPrhAN93g5HWF3JCkDsAlOHRipSbwZqycTjMueUB0&&variables_values=new order&route=otp&numbers=8949551146')
    console.log(out)
  }
}

onUpdated(async() => {
  if(user.value && user.value.uid){
    order.value.name = user.value.displayName;
    if(!defaultPhone.value){
      let data = await axios.get('https://droplet.netserve.in/pws-phone?uid='+user.value.uid).then(r => r.data);
      if(data && data[0]) {
        defaultPhone.value = data[0];
        order.value.phone_no = data[0].phone_no
      }
    }
  }
})

onMounted(async() => {
  products.value = await axios.get('https://droplet.netserve.in/pws-products').then(r => r.data)
  order.value.product_id = 1
})
</script>