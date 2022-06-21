<template>
<section class="p-4 bg-secondary w-full">
<div class="text-primary">
<p class="">{{moment(order?.ordered_at).format("ddd MMM DD, YYYY [at] HH:mm a")}}</p>
<hr />
<p>Hi, {{order?.name}}, <br />
Your order is comfirmed. Your order id is pws-{{order?.id}}.</p>
<p>Please pay &#8377;{{order?.product.price}}/- cash at the time of delivery!</p>
<p>You can track your order <router-link to="/track">here</router-link></p>

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
    order.value = await axios.get('https://droplet.netserve.in/pws-orders/'+id+'?expand=product').then(r => r.data)
    console.log(order.value)
})
</script>