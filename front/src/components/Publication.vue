<template>
    <div class="publication">
        <div class="publication__header">
            <img src="@/assets/profile-pic.png" alt="image de profil par défaut" class="publication__header__img">
            <p class="publication__header__auteur">
                {{publication.user.firstname}} {{publication.user.lastname}}
            </p>
        </div>
        <div class="publication__main">
            <p class="publication__main__texte">
                {{publication.title}} 
            </p>
            <img :src="publication.attachment" alt="Image ou gif publiée par un utilisateur" class="publication__main__img">
        </div>
        <div class="publication__footer">
            <p class="publication__footer__date">
                {{publication.createdAt | moment}}
            </p>
            <div class="publication__footer__social">
                <button v-if="publication.userId == auth.user.id || auth.user.roles == 'ROLE_ADMIN'" class="button-social" @click="deletePost(publication.id)">
                    <img src="@/assets/trash-alt-regular.png" alt="icone pour la suppression" class="icones-social">
                </button>
            </div>
        </div>
        <div class="publication__commentaires">
            <input type="text" v-model="textComment" v-on:keyup.enter="createComments" class="publication__commentaires__input" placeholder="un petit commentaire ?">
            <div id=bloc-commentaires v-for="comments in publication.comments" :key="comments.id" class="publication__commentaires__publies">
                <p class="commentaires__username">{{comments.user.firstname}} {{comments.user.lastname}}</p>
                <p class="commentaires__textUser">{{comments.text}}</p>
                <button v-if="comments.user.id == auth.user.id || auth.user.roles == 'ROLE_ADMIN'" class="button-social" @click="deleteComments(comments.id)">
                    <img src="@/assets/trash-alt-regular.png" alt="suppression d'un commentaire" class="commentaires__trash">
                </button>
            </div>
        </div>
    </div>

</template>
<script>
import moment from 'moment';
import axios from 'axios';
import {mapState} from 'vuex';
import authHeader from '../services/auth-header';
export default {
    name:"Publication",
    components: {},
    data(){
        return {
            textComment:""
        }
    },
    computed:{
        ...mapState(["publications","auth", ])
    },
    props: {
        publication: {
            type:Object,
            required: true
        },
        comments:{
            type:Object,

        }
    },
    methods: {
        deletePost(id) {
            axios
                .delete(`http://localhost:8081/api/publications/${id}`, {headers: authHeader()})
                .then(() => {
                    this.$store.dispatch('loadPublications')
                }) // ...Si non on envoi une erreur
                .catch(error => console.log(error));
        },
        createComments(){
            let dataComments = {
                text: this.textComment,
                publicationId: this.publication.id,
                userId: this.auth.user.id
            }
            axios
                .post('http://localhost:8081/api/commentaires',dataComments)
                .then((res)=>{
                    console.log(res)
                    this.textComment=""
                    this.$store.dispatch('loadPublications')
                });
        },
        deleteComments(id) {
            
            axios
                .delete(`http://localhost:8081/api/commentaires/${id}`,{headers: authHeader()})
                .then(() => {
                    this.$store.dispatch('loadPublications')
                }) // ...Si non on envoi une erreur
                .catch(error => console.log(error));
        },

    },
    filters:{
        moment: (date)=>{
            return moment(date).locale('fr').format('LLL');
        }
    }
};
</script>
<style lang="scss" >

.publication{
    background-color: #ffff;
    border: solid 2px #042255;
    border-radius: 10px;
    margin-bottom: 10px;
    &__header{
        display: flex;
        padding: 15px;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
        &__img{
            max-width: 50px;
        }
        &__auteur{
            font-size: 20px;
        }
    }
    &__main{
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        &__img{
            width: 100%;
        }
        &__texte{
            padding-left: 15px;
            text-align: left;
        }
    }
    &__footer{
        margin-top: 10px;
        padding: 5px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        &__social{
            display: flex;
            justify-content: space-evenly;
            width: 60%;
        }
        &__date{
            margin-top: 15px;
            padding-left: 15px;
            text-decoration:underline;

        }
    }
    &__commentaires{
        &__input{
            background-color: #fff;
            border: solid 2px #F64C71;
            border-radius: 25px;
            width: 90%;
            margin: 0 5% 5% 5%;
            padding-left: 5%;
        }
        &__publies{
            background-color: #CCD2DD;
            width: 90%;
            margin:0% 5% 5% 5%;
            border-radius: 25px;
            padding: 5%;
            position: relative;
        }
    }
}
.commentaires__username{
    color: #F64C71;
    margin: 0;
}
.commentaires__textUser{
    margin: 0;
}
.commentaires__trash{
    width: 20px;
    position: absolute;
    right: 10px;
    top: 10px;
}

.button-social{
    border: none;
    background-color: white;
    
}
.icones-social{
    max-width: 25px;
    max-height: 25px;
}
button{
    transition: none;
    transition-timing-function: cubic-bezier(.2, 3, .4, 1);
    &:hover{
    transform: none;
  }
}
    
</style>