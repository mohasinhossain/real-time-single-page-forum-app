<template>
    <div>
        <reply v-for="(reply, index) in content"
        :key="reply.id"
        :index="index"
        :data="reply"></reply>
    </div>
</template>

<script>
import Reply from './reply'
export default {
    components: {Reply},
    props: ['question'],
    data(){
        return{
            content: this.question.replies
        }
    },
    created() {
        this.listen()
    },
    methods: {
        listen(){
            EventBus.$on('newReply', (reply) => {
                this.content.unshift(reply)
            })

            EventBus.$on('deleteReply', (index) => {
                axios.delete(`/api/${this.question.slug}/reply/${this.content[index].id}`)
                .then(response => {
                    this.content.splice(index, 1)
                })
            })

            Echo.private('App.User.' + User.id())
                .notification((notification) => {
                //console.log(notification.type);
                this.content.unshift(notification.reply)
            });

            Echo.channel('DeleteReplyChannel')
            .listen('DeleteReplyEvent', (e) => {
                for(let index = 0; index < this.content.length; index++){
                    if(this.content[index].id == e.id){
                        this.content.splice(index, 1)
                    }
                }
            })
        }
    }
}
</script>

<style scoped>

</style>
