<script setup lang="ts">
import { ref, computed, onBeforeMount } from 'vue'
import { onBeforeRouteUpdate ,useRoute} from 'vue-router'
import { useMangaStore, MangaCollection } from '../stores/manga'
import { isApplicationError } from '../mixin/errorMessageMixing'
import MangaCard from '../components/MangaCard.vue'
import PaginatedContainer from '../components/PaginatedContainer.vue'

const mangaStore = useMangaStore()
const mangaCollection = ref<MangaCollection>({} as MangaCollection)
const mangas = computed(() => mangaCollection.value.mangas)
const pagination = computed(() => mangaCollection.value.pagination)
const loading = ref(true)
const route = useRoute()
const errorMessage = ref('')

async function getMangasAndUpdate(page = 1) {
  const result = await mangaStore.all(page)
  if(isApplicationError(result)) {
    errorMessage.value = result.message
  } else {
    mangaCollection.value = result
  }
  loading.value = false 
}

onBeforeMount(async () => getMangasAndUpdate(Number(route.query.page)))

onBeforeRouteUpdate(async (to, from) => {
  if (to.query.page !== from.query.page) { 
    getMangasAndUpdate(Number(to.query.page))
  } 
})

</script>

<template>
  <div class="text-center" v-if="loading">
    <div class="spinner-border" role="status">
        <span class="visually-hidden">Loading...</span>
    </div>
  </div>
  <PaginatedContainer v-else
   :page="pagination.number"
   :page-count="pagination.totalPages"
   :page-size = "pagination?.size"
   :total="pagination?.totalElements"
  >
    <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 g-3">
          <MangaCard v-for="manga in mangas" :key="manga.id"
          :id="manga.id"
          :title="manga.title"
          :cover="manga.cover"
          :number="manga.number"
          :price="manga.price"></MangaCard>
    </div>
  </PaginatedContainer>
</template>