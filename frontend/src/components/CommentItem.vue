<template>
    <div class="mb-6 flex items-center justify-between">
            <div class="flex items-center space-x-6">
                    <img :src="comment.created_by.get_avatar" class="w-[40px] rounded-full">
                                
                    <p><strong>{{ comment.created_by.name }}</strong></p>
            </div>

            <p class="text-gray-600">{{ comment.created_at_formatted }} ago</p>
        </div>

                        <p>
                            {{ comment.body }}
                        </p>

        <div class="my-6 flex justify-between">
                            
                            <!--div>
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                    <path stroke-linecap="round" stroke-linejoin="round" d="M12 6.75a.75.75 0 110-1.5.75.75 0 010 1.5zM12 12.75a.75.75 0 110-1.5.75.75 0 010 1.5zM12 18.75a.75.75 0 110-1.5.75.75 0 010 1.5z" />
                                </svg>   
                            </div-->   
                        </div>  
</template>
<script>
export default{
    props:{
        comment:Object
    },
}

</script>