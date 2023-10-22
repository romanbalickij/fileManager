<template>
    <modal :show="modelValue" @show="onShow"   max-width="sm">
           <div class="p-6">
            <h2 class="text-lg font-medium text-gray-900">
                Create new Folder
            </h2>
               <div class="mt-6"  >
                   <InputLabel for="folderName" value="folder Name"/>
                   <TextInput
                       ref="input"
                       type="text"
                       id="folderName"
                       v-model="form.name"
                       class="mt-1 block w-full"
                       placeholder="Folder Name"
                       :class="form.errors.name ? 'border-red-500 focus:border-red-500 focus:ring-red-500' : ''"

                       @keyup.enter="createFolder"
                   />
                   <InputError :message="form.errors.name" class="mt-2"></InputError>
               </div>
               <div class="mt-6 flex justify-end">
                   <SecondaryButton @click="closeModal">Cancel</SecondaryButton>
                   <PrimaryButton class="ml-3"
                                  :class="{ 'opacity-25': form.processing }"
                                  @click="createFolder"
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

import { router } from '@inertiajs/vue3'
import { useForm,  } from '@inertiajs/vue3'

 export default {
     components:{InputLabel, Modal, TextInput, InputError, SecondaryButton, PrimaryButton} ,
     data() {
         return {
             form: useForm({
                 name: null,
                 parent_id:null
             }),
         }
     },
     props:{
         modelValue:{
             type:Boolean
         }
     },

     methods:{
         createFolder() {
            this.form.parent_id = this.$page.props.folder.id

            this.form.post(route('folder.create'), {
                preserveScroll:true,
                onSuccess: () => {
                    this.closeModal()

                },
                onError:() =>{
                    this.$nextTick(() => {
                        this.$refs.input.focus()
                    });

                }
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
