<template>
    <modal :show="modelValue" @show="onShow"   max-width="sm">
           <div class="p-6">
            <h2 class="text-lg font-medium text-gray-900">
                Share files
            </h2>
               <div class="mt-6"  >
                   <InputLabel for="shareEmail" value="Enter Email Address"/>
                   <TextInput
                       ref="input"
                       type="text"
                       id="shareEmail"
                       v-model="form.email"
                       class="mt-1 block w-full"
                       placeholder="Enter Email Address"
                       :class="form.errors.email ? 'border-red-500 focus:border-red-500 focus:ring-red-500' : ''"

                       @keyup.enter="share"
                   />
                   <InputError :message="form.errors.email" class="mt-2"></InputError>
               </div>
               <div class="mt-6 flex justify-end">
                   <SecondaryButton @click="closeModal">Cancel</SecondaryButton>
                   <PrimaryButton class="ml-3"
                                  :class="{ 'opacity-25': form.processing }"
                                  @click="share"
                                  :disable="form.processing">Submit</PrimaryButton>
               </div>
           </div>
    </modal>
</template>
<script>
import Modal from "@/Components/Modal.vue";
import InputLabel from "@/Components/InputLabel.vue";
import TextInput from "@/Components/TextInput.vue";
import InputError from "@/Components/InputError.vue";
import SecondaryButton from "@/Components/SecondaryButton.vue";
import PrimaryButton from "@/Components/PrimaryButton.vue";
import {showSuccessNotification} from "@/event-bus.js";

import { router } from '@inertiajs/vue3'
import { useForm,  } from '@inertiajs/vue3'

 export default {
     components:{InputLabel, Modal, TextInput, InputError, SecondaryButton, PrimaryButton} ,
     data() {
         return {
             form: useForm({
                 email: null,
                 ids:[],
                 parent_id:null,
                 all:false
             }),
         }
     },
     props:{
         modelValue:{
             type:Boolean
         },
         allSelected: {
             type: Boolean,
             required: false,
             default: false
         },
         selectedIds: {
             type: Array,
             required: false
         }
     },

     methods:{
         share() {
             this.form.parent_id = this.$page.props.folder.id

             if (this.allSelected) {
                 this.form.all = true;
                 this.form.ids = [];
             }  else {
                 this.form.ids = this.selectedIds
             }
             const email = this.form.email
             this.form.post(route('file.share'), {
                 preserveScroll: true,
                 onSuccess: () => {
                     this.closeModal()
                     this.form.reset();
                     // Show success notification
                     showSuccessNotification(`Selected files will be shared to "${email}" if the emails exists in the system`)
                 },
             })
         },
         closeModal() {
              this.$emit('update:modelValue')
              this.form.clearErrors();
              this.form.reset();

         },
         onShow() {
             this.$nextTick(() => {
                 this.$refs.input.focus()
             });

         }
     }
 }
</script>
