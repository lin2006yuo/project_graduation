<template>
    <div class="home">
        <navgator></navgator>
        <keep-alive >
            <router-view></router-view>
        </keep-alive>
        <basic-vue-chat
            v-if="login"
            class="chat" 
            ref="vuechat"
            :initial-feed="msgList"
            @newOwnMessage="send"
            @click="handleClick"
            @toggleShow="toggleShow"
            @delete="handleDelete"
            :chat-list="chatList"
            :send-flag="login"
            :title="title"
            :show="show"
        />
        <!-- <footer></footer> -->
    </div>
</template>

<script type="text/ecmascript-6">
import Navgator from 'components/navigator/navigator'
import BasicVueChat from 'components/basic-vue-chat/BasicVueChat.vue'
import { mapGetters, mapActions, mapMutations } from 'vuex'
import * as types from '@/store/front/mutation-types'
import moment from 'moment'

export default {
    data() {
        return {
            feed: [
                {
                    id: 0,
                    author: 'Person',
                    contents: 'hi there',
                    date: '16:30'
                },
                {
                    id: 1,
                    author: 'Me',
                    contents: 'hello',
                    date: '16:30'
                }
            ],
            login: false
        };
    },
    computed: {
        ...mapGetters([
            'studentInfo',
            'companyInfo',
            "chatList",
            "currentChator",
            "message",
            "msgList",
            "show",
            "role"
        ]),
        title() {
            return this.currentChator.name ? this.currentChator.name : this.currentChator.companyName
        }
    },
    mounted() {
            setTimeout(() => {
                if(this.studentInfo._id || this.companyInfo._id) {
                    this.login = true
                    this.setLogin(this.login)
                    if(this.login) {
                        let from,role
                        if(this.studentInfo._id) {
                            from = this.studentInfo._id,
                            role = 'student'
                            this.setRole(role)
                        } else {
                            from = this.companyInfo._id
                            role = 'company'
                            this.setRole(role)
                        }
                        this.initChatList({from, role})
                    }
                }
            }, 200)
    },
    methods: {
        ...mapActions([
           'sendMsg',
           'getMsgList',
           'initChatList',
           "deleteChator"
        ]),
        ...mapMutations({
            'setCurrentChator': types.SET_CURRENT_CHATOR,
            'setLogin': types.SET_LOGIN,
            'setRole': types.SET_ROLE,
            'showChat': types.SET_CHAT
        }),
        send(message) {
            if(!this.login) return this.$message('请登录...')
            const from = this.studentInfo._id ? this.studentInfo._id : this.companyInfo._id
            const to = this.currentChator._id
            this.sendMsg({from, to, content: message.contents, message})
        },
        handleClick(item, index) {
            const chator = this.chatList[index]
            this.$set(chator, 'dot', false)
            this.getMsgList(item)
        },
        toggleShow() {
            this.showChat(!this.show)
        },
        handleDelete(item) {
            const from = this.role === 'student' ? this.studentInfo._id : this.companyInfo._id
            const to = item._id
            this.deleteChator({from, to})
        }
    },
    components: {
        Navgator,BasicVueChat
    },
    watch: {
        // 'message.create_time'(val) {
        //     const msg = {
        //         id: 1,
        //         author: 'Person',
        //         contents: this.message.content,
        //         date: moment(val.create_time).format("HH:mm:ss")
        //     }
        //     this.feed.push(msg)
        //     this.$refs.vuechat.toBottom()
        // },
        'msgList'(val) {
                this.$nextTick(() => {
                    this.$refs.vuechat.toBottom()
                })
        }
    }
};
</script>

<style scoped lang="stylus">
.home
    height 100%
    background-color #fafafa
.chat 
    position fixed
    bottom 0
    right 0
    z-index 500
    // transform translate3d(0, 90%, 0)
footer {
    margin-top 50px
    height 100px
}
        
</style>
