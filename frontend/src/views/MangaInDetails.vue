<script setup lang="ts">
import { onBeforeMount, ref } from 'vue';
import { useRoute, onBeforeRouteUpdate } from 'vue-router'
import { useMangaStore, Manga} from '../stores/manga'
import { imgURL } from '../mixin/mangaMixing'
import { isApplicationError } from '../mixin/errorMessageMixing'
import CommentsContainer from '../components/Comment/Container.vue'

const mangaStore = useMangaStore()

const route = useRoute()
const id = Number(route.params.id)
const manga = ref<Manga>({} as Manga)  
const loading = ref(true)
const errorMessage = ref('')

async function getMangaAndUpdate(id: number) {
    const result = await mangaStore.get(id)
    if (isApplicationError(result)) {
        errorMessage.value = result.message
    } else {
        manga.value = result
    }
    loading.value = false 
}

onBeforeMount( async () => {
    getMangaAndUpdate(id)
})


onBeforeRouteUpdate( async (to, from) => {
    if (to.params.id !== from.params.id) {
        getMangaAndUpdate(id)
    } 
})

</script>

<template>
    <div class="text-center" v-if="!manga.id">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading...</span>
        </div>
    </div>
    <div class="row justify-content-center" v-else>
        <div class="col-lg-8 col-sm-12">
            <div class="card mb-3">
                <div class="row g-0">
                    <div class="col-md-4">
                        <img :src="imgURL(manga.cover)" class="w-100 rounded-start" :alt="manga.cover">
                    </div>
                    <div class="col-md-8">
                        <div class="card-body">
                            <h5 class="card-title">{{manga.title}}</h5>
                            <hr>
                            <div class="text-start">
                                <p class="card-text">Volume: {{manga.number}}</p>
                                <p class="card-text"><strong>Preço: <small class="text-danger">{{manga.price}}</small></strong></p>
                                <p class="card-text"><strong>Sumário:</strong> <br>{{manga.summary}}</p>
                            </div>
                        </div>
                    </div>
                </div>
                
            </div>
        </div>

        <div class="row">
            <CommentsContainer class="col-12" :comments="manga.comments"></CommentsContainer>
        </div>
    </div>
</template>