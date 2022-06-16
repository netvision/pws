<template>
<section class="p-4 bg-secondary w-full">
<div class="text-primary">
<p>Hi, {{order?.name}}, <br />
Your order is comfirmed. Please pay Rs. 500/- cash at the time of delivery!</p>
<p>You can track your order <router-link to="/track">here</router-link></p>
<pre>{{moment(order?.ordered_at).format("ddd MMM DD, YYYY [at] HH:mm a")}}</pre>
</div>
</section>
</template>
<script setup>
import {ref, onMounted} from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';
import moment from 'moment'

const route = useRoute()
const id = route.params.id
const order = ref()
onMounted(async() => {
    order.value = await axios.get('https://droplet.netserve.in/pws-orders/'+id).then(r => r.data)
    console.log(order.value)
})
</script>