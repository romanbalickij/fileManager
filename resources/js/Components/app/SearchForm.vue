<template>
    <div class="w-[600px] h-[80px] flex items-center">
       <TextInput
           type="text" class="block w-full mr-2"
                   v-model="search"
           @keyup.prevent.enter="onSearch"
       />
    </div>
    <div class="flex-1 flex flex-col overflow-hidden">
        <slot/>
    </div>
</template>

<script>
import TextInput from "@/Components/TextInput.vue";
import { router } from '@inertiajs/vue3'
export default {
    components: {TextInput},

    data() {
        return {
                search:'',
                params :null

        }
    },

    methods:{
        onSearch() {


           this.params.set('search', this.search)

           router.get(window.location.pathname + `?${this.params.toString()}`)
        }
    },

    mounted() {
        this.params = new URLSearchParams(window.location.search)
        this.search = this.params.get('search')
    }
}

</script>

<style scoped>

</style>
