<template>
    <div class="max-w-7xl mx-auto grid grid-cols-4 gap-4">
        <div class="main-left col-span-1">
                    <div class="p-4 bg-white border border-gray-200 text-center rounded-lg">
                        <img :src="user.get_avatar" class="mb-6 rounded-full">
                        <p><strong>{{ user.name }}</strong></p>

                        <div class="mt-6 flex space-x-8 justify-around">
                            <p class="text-xs text-gray-500">{{ user.friends_count }} friends</p>
                            <p class="text-xs text-gray-500">{{ user.posts_count }} posts</p>
                        </div>
                        <!--div class="mt-6" v-if="userStore.user.id!=user.id & ">
                            <button class="inline-vlock py-4 px-6 bg-red-600 text-sm text-white rounded-lg" @click="SendRequest">Request</button>
                        </div-->
                    </div>
        </div>
        <div class="main-center col-span-2 space-y-4">
            <div class="p-4 bg-white border border-gray-200 rounded-lg"
                    v-if="friendrequests.length & user.id==userStore.user.id"
                    >
                    <h2>Friend Request</h2>
                       <div 
                        v-for="friendrequest in friendrequests"
                        v-bind:keys="friendrequest.id"
                        class="p-4 text-center bg-gray-100 rounded-lg">
                            <img :src="friendrequest.created_by.get_avatar" class="mb-6 mx-auto rounded-full">
                        
                            <p><strong>
                                <RouterLink :to="{name:'profile',params:{'id':friendrequest.created_by.id}}">{{ friendrequest.created_by.name }}</RouterLink>
                            </strong></p>

                            <div class="mt-6 flex space-x-8 justify-around">
                                <p class="text-xs text-gray-500">{{ friendrequest.created_by.friends_count }} friends</p>
                                <p class="text-xs text-gray-500">{{ friendrequest.created_by.posts_count }} posts</p>
                            </div>
                        
                        <div class="mt-6 space-x-4 ">
                            <button class="inline-block py-4 px-6 bg-purple-600 text-white rounded-lg mx-auto " @click="HandleRequest('Accepted',friendrequest.created_by.id)">accept</button>
                            <button class="inline-block py-4 px-6 bg-red-600 text-white rounded-lg" @click="HandleRequest('Rejected',friendrequest.created_by.id)">reject</button>
                        </div>
                        </div>
                        <hr>
                    </div>
           
            <div class="p-4 bg-white border border-gray-200 rounded-lg grid grid-cols-2 gap-4"
                    v-if="friends.length"
                    >
                    <h2>Friends</h2>
                        <div 
                        v-for="user in friends"
                        v-bind:keys="user.id"
                        class="p-4 text-center bg-gray-100 rounded-lg">
                            <img :src="user.get_avatar" class="mb-6 rounded-full">
                        
                            <p><strong>
                                <RouterLink :to="{name:'profile',params:{'id':user.id}}">{{ user.name }}</RouterLink>
                            </strong></p>
                                <p class="text-xs text-gray-500">{{ user.friends_count }} friends</p>
                                <p class="text-xs text-gray-500">120 posts</p>
                            </div>
                        </div>
                    </div>
        
    <div class="main-right col-span-1 space-y-4">
          <youmknow/>
          <Trends/>
    </div>
</div>
</template>
<script>
import youmknow from '@/components/youmknow.vue';
import Trends from '@/components/Trends.vue';
import axios from 'axios';
import { useUserStore } from '@/stores/user'

import FeedItem from '@/components/FeedItem.vue';
export default{
    name:'FriendsView',
    setup() {
        const userStore = useUserStore()
        return {
            userStore
        }
    },
    components:{
        youmknow,
        Trends,
    },
    data(){
        return{
            user:{},
            friends:[],
            friendrequests:[],
        }
    },
    mounted(){
        this.getfriends()
    },
    methods:{  
        getfriends(){
            axios
            .get(`/api/friends/${this.$route.params.id}/`)
            .then(response=>{
                console.log('data',response)
                this.friendrequests=response.data.requests
                this.friends=response.data.friends
                this.user=response.data.user
            })
            .catch(error=>{
                console.log('error',error)
            })
        },
        HandleRequest(status,pk){
            axios
            .post(`/api/friends/${pk}/${status}/`)
            .then(response=>{
                console.log('data',response.data)
                this.friendrequests.unshift()
            })
            .catch(error=>{
                console.log('error',error)
            })
        }
    }
}
</script>