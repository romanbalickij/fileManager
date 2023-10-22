<template>
    <AuthenticatedLayout>
        <nav class="flex items-center justify-end p-1 mb-3">

            <div>
               <DeleteForeverButton
                   :all-selected="allSelected"
                   :selected-ids="selectedIds"
               >

               </DeleteForeverButton>
               <RestoreFilesButton
                   :all-selected="allSelected"
                   :selected-ids="selectedIds"
               >
               </RestoreFilesButton>
            </div>
        </nav>

        <div class="flex-1 overflow-auto">

            <table  class="min-w-full">
                <thead class="bg-gray-100 border-b">
                <th  class="text-sm font-medium text-gray-900 px-6 py-4 text-left w-[30px] max-w-[30px] pr-0">
                    <Checkbox @change="onSelectedAll" v-model:checked="allSelected"/>
                </th>
                <tr>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                        Name
                    </th>
                    <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                        Path
                    </th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="file of allFiles.data" :key="file.id"
                    class="bg-white border-b transition duration-300 ease-in-out hover:bg-blue-100 cursor-pointer"
                    :class="(selected[file.id] || allSelected) ? 'bg-blue-50' : 'bg-white'"
                    @click="$event => toggleFileSelected(file)"
                >

                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 w-[30px] max-w-[30px] pr-0">
                        <Checkbox
                            v-model="selected[file.id]"
                            :checked="selected[file.id] || allSelected"
                            @change="$event => onSelecteChecboxChange(file)"
                        />
                    </td>

                    <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 flex items-center">
                        <FileIcon :file="file"/>
                        {{file.name}}
                    </td>
                    <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                        {{file.path}}
                    </td>
                </tr>
                </tbody>
            </table>

            <div v-if="!allFiles.data.length" class="py-8 text-center text-lg text-gray-400">
                The is not  data in this folder
            </div>
            <div ref="loadMoreIntersect"></div>
        </div>
    </AuthenticatedLayout>
</template>
<script>
import { router } from '@inertiajs/vue3'
import {Link} from '@inertiajs/vue3'
import {HomeIcon} from '@heroicons/vue/20/solid'

import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import FileIcon from "@/Components/app/FileIcon.vue";
import {httpGet} from "@/Helper/http-helper";
import Checkbox from "@/Components/Checkbox.vue";
import DeleteFilesButton from "@/Components/app/DeleteFilesButton.vue";
import DownloadFilesButton from "@/Components/app/DownloadFilesButton.vue";
import RestoreFilesButton from "@/Components/app/RestoreFilesButton.vue";
import DeleteForeverButton from "@/Components/app/DeleteForeverButton.vue";

export default {
    components: {
        DeleteForeverButton,
        RestoreFilesButton,
        DownloadFilesButton, DeleteFilesButton, Checkbox, FileIcon, AuthenticatedLayout, Link, HomeIcon},

    data() {
        return {
            allSelected:false,
            selected: {},

            allFiles: {
                data:this.files.data,
                next:this.files.links.next
            }
        }
    },

    props:{
        files:Object,
        folder:{
            type:Object
        },
        ancestors:{

        },

    },

    computed: {
        selectedIds() {
            return Object.entries(this.selected)
                .filter((el) => el[1])
                .map(id => id[0])
        }
    },

    updated() {
        this.allFiles = {
            data:this.files.data,
            next:this.files.links.next
        }

    },

    mounted() {
        const observer = new IntersectionObserver((entries) =>  entries.forEach(entry => entry.isIntersecting && this.loadMore()), {
            rootMargin: '-250px 0px 0px 0px'
        })


        observer.observe(this.$refs.loadMoreIntersect)
    },

    methods:{
        loadMore() {
            if(this.allFiles.next === null) {
                return
            }

            httpGet(this.allFiles.next).then(response =>{
                this.allFiles.data = [...this.allFiles.data, ...response.data]
                this.allFiles.next = response.links.next

            })

        },
        onSelectedAll() {
            this.allFiles.data.forEach(f =>{
                this.selected[f.id] = this.allSelected;
            })
        },
        toggleFileSelected(file) {
            this.selected[file.id] = !this.selected[file.id]
            this.onSelecteChecboxChange(file)
        },
        onSelecteChecboxChange(file) {
            if(!this.selected[file.id]) {
                this.allSelected = false
            }else {
                // let checked = true;
                //
                // for(let file of this.allFiles.data) {
                //     if(!this.selected[file.id]) {
                //         checked = false;
                //         break;
                //     }
                // }
                //
                // this.allSelected = checked
            }
        },

        onDelete() {

            this.allSelected = false
            this.selectedIds = {}
        }
    }
}
</script>
