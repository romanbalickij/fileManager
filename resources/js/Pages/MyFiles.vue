<template>
<AuthenticatedLayout>

    <nav class="flex items-center justify-between p-1 mb-3">
        <ol class="inline-flex items-center space-x-1 md:space-x-3">
            <li v-for="ans of ancestors.data" :key="ans.id" class="inline-flex items-center">
                <Link v-if="!ans.parent_id" :href="route('myFiles')"     class="inline-flex items-center text-sm font-medium text-gray-700 hover:text-blue-600 dark:text-gray-400 dark:hover:text-white">
                    <HomeIcon class="w-4 h-4"/>
                    My Files
                </Link>
                <div v-else class="flex items-center">
                    <svg aria-hidden="true" class="w-6 h-6 text-gray-400" fill="currentColor" viewBox="0 0 20 20"
                         xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd"
                              d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                              clip-rule="evenodd"></path>
                    </svg>
                    <Link :href="route('myFiles', {folder: ans.path})"
                          class="ml-1 text-sm font-medium text-gray-700 hover:text-blue-600 md:ml-2 dark:text-gray-400 dark:hover:text-white">
                        {{ ans.name }}
                    </Link>
                </div>
            </li>
        </ol>

        <div class="flex">
            <label class="flex items-center mr-3">
                Only favorites
                <Checkbox @change="showOnlyFavourites" v-model:checked="onlyFavourites" class="ml-2"/>
            </label>
            <ShareFilesButton :all-selected="allSelected" :selected-ids="selectedIds"></ShareFilesButton>
            <DownloadFilesButton :all="allSelected" :ids="selectedIds" class="mr-2"></DownloadFilesButton>
            <DeleteFilesButton :delete-all="allSelected" :delete-ids="selectedIds" @delete="onDelete"/>
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

                </th>
                <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                    Name
                </th>
                <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                    Owner
                </th>
                <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                    Last Modified
                </th>
                <th class="text-sm font-medium text-gray-900 px-6 py-4 text-left">
                    Size
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="file of allFiles.data" :key="file.id"
                class="bg-white border-b transition duration-300 ease-in-out hover:bg-blue-100 cursor-pointer"
                :class="(selected[file.id] || allSelected) ? 'bg-blue-50' : 'bg-white'"
                @dblclick="openFolder(file)"
                @click="$event => toggleFileSelected(file)"
            >

                <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 w-[30px] max-w-[30px] pr-0">
                    <Checkbox
                        v-model="selected[file.id]"
                        :checked="selected[file.id] || allSelected"
                        @change="$event => onSelecteChecboxChange(file)"
                    />
                </td>
                <td   class="px-6 py-4 max-w-[40px] text-sm font-medium text-gray-900 text-yellow-500">
                    <div @click.stop.prevent="addRemoveFavourite(file)">
                        <svg v-if="!file.is_favourite" xmlns="http://www.w3.org/2000/svg" fill="none"
                             viewBox="0 0 24 24" stroke-width="1.5"
                             stroke="currentColor" class="w-6 h-6">
                            <path stroke-linecap="round" stroke-linejoin="round"
                                  d="M11.48 3.499a.562.562 0 011.04 0l2.125 5.111a.563.563 0 00.475.345l5.518.442c.499.04.701.663.321.988l-4.204 3.602a.563.563 0 00-.182.557l1.285 5.385a.562.562 0 01-.84.61l-4.725-2.885a.563.563 0 00-.586 0L6.982 20.54a.562.562 0 01-.84-.61l1.285-5.386a.562.562 0 00-.182-.557l-4.204-3.602a.563.563 0 01.321-.988l5.518-.442a.563.563 0 00.475-.345L11.48 3.5z"/>
                        </svg>
                        <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"
                             class="w-6 h-6">
                            <path fill-rule="evenodd"
                                  d="M10.788 3.21c.448-1.077 1.976-1.077 2.424 0l2.082 5.007 5.404.433c1.164.093 1.636 1.545.749 2.305l-4.117 3.527 1.257 5.273c.271 1.136-.964 2.033-1.96 1.425L12 18.354 7.373 21.18c-.996.608-2.231-.29-1.96-1.425l1.257-5.273-4.117-3.527c-.887-.76-.415-2.212.749-2.305l5.404-.433 2.082-5.006z"
                                  clip-rule="evenodd"/>
                        </svg>
                    </div>
                </td>
                <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 flex items-center">
                    <FileIcon :file="file"/>
                    {{file.name}}
                </td>
                <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                    {{file.path}}
                </td>
                <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                    {{file.owner}}
                </td>
                <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                    {{file.updated_at}}
                </td>
                <td  class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                    {{file.size}}
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
import {emitter, ON_SEARCH, showSuccessNotification} from "@/event-bus.js";

import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import FileIcon from "@/Components/app/FileIcon.vue";
import {httpGet, httpPost} from "@/Helper/http-helper";
import Checkbox from "@/Components/Checkbox.vue";
import DeleteFilesButton from "@/Components/app/DeleteFilesButton.vue";
import DownloadFilesButton from "@/Components/app/DownloadFilesButton.vue";
import StarFilesButton from "@/Components/app/StarFilesButton.vue";
import ShareFilesButton from "@/Components/app/ShareFilesButton.vue";

export default {
    components: {
        ShareFilesButton,
        StarFilesButton,
        DownloadFilesButton, DeleteFilesButton, Checkbox, FileIcon, AuthenticatedLayout, Link, HomeIcon},

    data() {
        return {
            allSelected:false,
            onlyFavourites:false,
            params:null,
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


        this.params = new URLSearchParams(window.location.search)
        this.onlyFavourites = this.params.get('favourites') == 1


    },

    methods:{
        showOnlyFavourites() {
         //   this.onlyFavourites = !this.onlyFavourites

            if (this.onlyFavourites) {
                this.params.set('favourites', 1)
            } else {
                this.params.delete('favourites')
            }

            router.get(window.location.pathname+'?'+ this.params.toString())

        },
        loadMore() {
              if(this.allFiles.next === null) {
                  return
              }

            httpGet(this.allFiles.next).then(response =>{
                this.allFiles.data = [...this.allFiles.data, ...response.data]
                this.allFiles.next = response.links.next

            })

        },
        openFolder(file) {
            if(!file.is_folder) {
                return
            }
            router.visit(route('myFiles', {folder:file.path}))

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
        },
        addRemoveFavourite(file) {

            httpPost(route('file.addToFavourites'), {id: file.id})
                .then(() => {
                    file.is_favourite = !file.is_favourite
                    showSuccessNotification('Selected files have been added to favourites')
                })
                .catch(async (er) => {
                    console.log(er.error.message);
                })
        }
    }
}
</script>
