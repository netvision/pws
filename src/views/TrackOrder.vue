<template>
<section class="p-4 bg-secondary w-full">
<div class="text-primary">
    <pre>{{order}}</pre>
</div>
</section>

</template>
<script setup>
import {ref, onMounted} from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

const route = useRoute()
const id = route.params.id
const order = ref()
onMounted(async() => {
    order.value = await axios.get('https://droplet.netserve.in/pws-orders/'+id+'?expand=product,statuses').then(r => r.data)
    console.log(order.value)
})
</script>