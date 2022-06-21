<template>
<section class="p-4 bg-secondary w-full">
<div class="text-primary">
    <form>
    <div class="form-group mb-6">
      <label for="order_code" class="form-label inline-block mb-2 text-gray-700">Order Code</label>
      <input type="text" v-model="orderCode" class="form-control
        block
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
        focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none" id="order_code"
        aria-describedby="emailHelp" placeholder="Enter Code">
    </div>
    
    <button type="submit" class="
      px-6
      py-2.5
      bg-blue-600
      text-white
      font-medium
      text-xs
      leading-tight
      uppercase
      rounded
      shadow-md
      hover:bg-blue-700 hover:shadow-lg
      focus:bg-blue-700 focus:shadow-lg focus:outline-none focus:ring-0
      active:bg-blue-800 active:shadow-lg
      transition
      duration-150
      ease-in-out" @click="getOrder">Submit</button>
  </form>
  <div class="mt-6" v-if="order">
    <p>Ordered At: {{moment(order?.ordered_at).format("ddd MMM DD, YYYY [at] HH:mm a")}}</p>
    <p>Current Status: {{order.current_status}}</p>
    <p>Status Log:</p>
    <ul class="list-disc list-inside">
        <li v-for="status in order.statuses">{{status.status}} ({{status.status_note}}) at {{moment(status.updated_at).format("ddd MMM DD, YYYY [at] HH:mm a")}}</li>
    </ul>
  </div>
</div>
</section>

</template>
<script setup>
import {ref} from 'vue';
import axios from 'axios';
import moment from 'moment'
const orderCode = ref()
const order = ref()

const getOrder = async(e) => {
    e.preventDefault();
    let id = orderCode.value.split("-")[1]
    console.log(id)
    order.value = await axios.get('https://droplet.netserve.in/pws-orders/'+id+'?expand=product,statuses').then(r => r.data)
}

</script>