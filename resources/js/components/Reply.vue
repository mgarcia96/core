<template>
    <div class="pt-2" v-if="!loading" v-cloak>

        <transition name="fade">
            <div v-if="success" class="text-md text-center p-4 bg-green-100 text-green-700 shadow rounded">
                {{ chatterTrans.get('chatter.messages.reply_posted') }}.
            </div>
        </transition>

        <div v-if="auth && !success" class="w-full text-center">
            <div v-if="errors" class="p-2 mb-2 rounded shadow bg-red-200 text-red-700">
                <div v-for="(value, key, index) in errors">{{ value[0] }}</div>
            </div>

            <editor :event="event"></editor>

            <div class="p-2">
                <div @click="save()" class="w-32 inline-block bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full text-center cursor-pointer">
                  {{ chatterTrans.get('chatter.messages.submit') }}
                </div>
            </div>
        </div>

        <div v-if="!auth">
            <div class="text-md text-center p-4">{{ chatterTrans.get('chatter.messages.please') }} <a class="font-bold text-blue-700" href="/login">{{ chatterTrans.get('chatter.messages.login') }}</a> {{ chatterTrans.get('chatter.messages.or') }} <a class="font-bold text-blue-700" href="/register">{{ chatterTrans.get('chatter.messages.signup') }}</a> {{ chatterTrans.get('chatter.messages.leave_response') }}.</div>
        </div>
    </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex';
import Editor from './editor/Editor'
import { events } from '../events'
import axios from 'axios'

export default {
    components: {
        Editor
    },
    computed: {
        ...mapGetters([
            'auth',
            'loading'
        ])
    },
    props: {
        discussion: Object
    },
    data() {
        return {
            success: false,
            event: events.POST_CONTENT_UPDATE,
            body: null,
            errors: null
        }
    },
    mounted() {
        var self = this

        window.ChatterEvents.$on(events.POST_CONTENT_UPDATE, html => {
            self.body = html
        })
    },
    methods: {
        ...mapMutations([
            'showAlert'
        ]),
        save() {
            var self = this

            axios.post('/api/chatter/post', {
                    body: self.body,
                    discussion_id: this.discussion.id
                })
                .then(response => {
                    self.body = null;
                    self.success = true;

                    window.ChatterEvents.$emit(events.ADD_POST_TO_DISCUSSION, response.data.data)
                })
                .catch(error => {
                    let response = error.response

                    if (422 === response.status) {
                        this.errors = response.data.errors
                    } else {
                        this.showAlert({
                            title: 'Error',
                            text: response.data.message,
                            color: 'red'
                        })
                    }

                    console.error(error)
                })

        }
    }
}
</script>