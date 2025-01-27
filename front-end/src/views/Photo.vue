<template>
    <div class="photo">
        <Updater :photo="photo" :show="show" @close="close" @updateFinished="updateFinished" />

        <h1>{{this.photo.title}}</h1>
        <h3>{{this.user.username}}</h3>
        <p>{{formatDate(photo.created)}}</p>

        <div class="mb-2" v-if="showEdit()">
            <button class="btn btn-outline-primary me-2" to="/profile" @click="toggleUpdater(true)">Edit</button>
            <router-link to="/profile">
                <button class="btn btn-outline-danger me-2" to="/profile" @click="deletePhoto()">Delete</button>
            </router-link>
        </div>

        <img :src="photo.path" class="img-fluid" alt="Responsive image">
        <p class="mt-3 mb-4 px-2">{{photo.description}}</p>

        <form class="mb-4" v-if="this.$root.$data.user">
            <legend>Comment</legend>
            <div class="mb-3">
                <textarea class="form-control" placeholder="Comment" v-model="newComment"></textarea>
            </div>
            <div class="alert alert-danger mt-3 mb-3 mb-0 text-start" role="alert" v-if="error">{{this.error}}</div>
            <button class="btn btn-primary" @click.prevent="stageComment()">Submit</button>
        </form>

        <div class="mb-5">
            <div v-for="comment in comments" v-bind:key="comment._id">
                <hr>
                <div class="comment">
                    <p class="mb-2 text-bold">
                        <strong>{{comment.user.firstName + " " + comment.user.lastName}}</strong>
                    </p>
                    <p class="px-2 mb-1">{{comment.comment}}</p>
                    <small class="text-muted">{{formatDate(comment.created)}}</small>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Updater from '@/components/Updater.vue';
import axios from 'axios';
import moment from 'moment';

export default {
    name: 'Photo',

    components: {
        Updater
    },

    data() {
        return {
            photo: Object,
            comments: Array,
            newComment: '',
            user: {
                id: ''
            },
            show: false,
            error: ''
        }
    },

    async created() {
        this.getPhoto();
        this.getComments();
        this.getUser();
    },

    methods: {
        async getPhoto() {
            try {
                let response = await axios.get(`/api/photos/${this.$route.params.id}`);
                this.photo = response.data;
                this.user = this.photo.user;
                this.user.id = this.user._id;
            } catch (error) {  /* console.log(error); */  }
        },

        async getComments() {
            try {
                let response = await axios.get(`/api/comments/${this.$route.params.id}`);
                this.comments = response.data;
            } catch (error) {  /* console.log(error); */  }
        },

        async getUser() {
            try {
                let response = await axios.get('/api/users');
                this.$root.$data.user = response.data.user;
            } catch (error) {  this.$root.$data.user = null;  }
        },

        stageComment() {
            this.submitComment();
            this.newComment = '';
            this.getComments();
        },

        async submitComment() {
            this.error = '';
            if (this.newComment != '') {
                try {
                    await axios.post(`/api/comments/${this.$route.params.id}`, {
                        comment: this.newComment,
                        photo: this.$route.params.id,
                        user: this.$root.$data.user
                    });
                } catch (error) {  /* console.log(error); */  }
            } else {
                this.error = "Error: please provide a comment";
            }
        },
        
        formatDate(date) {
            if (moment(date).diff(Date.now(), 'days') < 15) {
                return moment(date).fromNow();
            } else {
                return moment(date).format('d MMMM YYYY');
            }
        },

        async deletePhoto() {
            try {
                await axios.delete(`/api/photos/${this.$route.params.id}`);
            } catch (error) { /* console.log(error); */  }
        },

        async updateFinished() {
            this.show = false;
            this.getPhoto();
        },

        toggleUpdater(value) {
            this.show = value;
        },

        close() {
            this.show = false;
        },

        showEdit() {
            if (this.$root.$data.user === null) {
                return false;
            } else {
                if (this.$root.$data.user._id === this.user.id) {
                    return true;
                } else {
                    return false;
                }
            }
        }
    }
}
</script>