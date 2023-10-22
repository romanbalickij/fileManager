<template>
<div class="h-screen bg-gray-50 flex w-full gap-4">
  <Navigation/>

    <main
        @drop.prevent="handlerDrop"
        @dragover.prevent="onDragOver"
        @dragleave.prevent="onDragLeave"
        :class="dragOver ? 'dropzone' : ''"
        class="flex flex-col flex-1 px-4 overflow-hidden">
        <template v-if="dragOver" class="text-gray-500 text-center py-8 text-sm">
            Drop files
        </template>
        <template v-else>
            <div class="flex items-center justify-between w-full">
                <SearchForm/>
                <UserSettingsDropdown/>
            </div>
            <div class="flex-1 flex flex-col overflow-hidden">
                <slot/>
            </div>
        </template>
    </main>
</div>
    <ErrorDialog/>
    <FormProgress :form="fileUploadForm"/>
    <Notification />
</template>

<script>
import { useForm } from '@inertiajs/vue3'

import {emitter, FILE_UPLOAD_STARTED, showErrorDialog, SHOW_ERROR_DIALOG, showSuccessNotification} from "@/event-bus";
import Navigation from "@/Components/app/Navigation.vue";
import SearchForm from "@/Components/app/SearchForm.vue";
import UserSettingsDropdown from "@/Components/app/UserSettingsDropdown.vue";
import FormProgress from "@/Components/app/FormProgress.vue";
import ErrorDialog from "@/Components/ErrorDialog.vue";
import Notification from "@/Components/Notification.vue";

export default {
    components: {ErrorDialog, FormProgress, UserSettingsDropdown, SearchForm, Navigation, Notification},

    data() {
        return {
            dragOver:false,
            fileUploadForm: useForm({
                files: [],
                parent_id:null,
                relative_paths:[]
            }),
        }
    },

    mounted() {
        emitter.on(FILE_UPLOAD_STARTED, this.uploadFiles)
    },

    methods:{
        uploadFiles(files) {
            this.fileUploadForm.parent_id = this.$page.props.folder.id
            this.fileUploadForm.files = files
            this.fileUploadForm.relative_paths = [...files].map(f => f.webkitRelativePath)

            this.fileUploadForm.post(route('file.store'), {
                onSuccess:() =>{
                    showSuccessNotification(`${files.length} files uploaded`)
                },
                onError:errors =>{
                    let message = '';

                    if(Object.keys(errors).length > 0) {
                        message = errors[Object.keys(errors)[0]]
                    }else {
                        message = 'Error during upload. Please try agen'
                    }

                    showErrorDialog(message)
                },
                onFinish:() =>{
                    this.fileUploadForm.clearErrors()
                    this.fileUploadForm.reset()
                }
            })
        },
        handlerDrop(e) {
          this.dragOver = false
          const files = e.dataTransfer.files

            if(!files.length) {
                return
            }
            this.uploadFiles(files)

        },

        onDragOver(e) {
            this.dragOver = true

        },
        onDragLeave(e) {
            this.dragOver = false

        }
    }
}
</script>

<style scoped>
    .dropzone {
        width: 100%;
        height: 100%;
        color: #8d8d8d;
        border: 2px dashed gray;
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>
