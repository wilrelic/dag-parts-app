<script setup>
// VARIABLES & OBJECTS
const config = useRuntimeConfig()
const rapidKey = config.secret
const amazonUrl = 'https://amazon24.p.rapidapi.com/api/product?'
const ebayUrl = 'https://ebay-data-scraper.p.rapidapi.com/products?'
const aliUrl = 'https://magic-aliexpress1.p.rapidapi.com/api/products/search?'
const searchParams = ref("")
const amazonProducts = ref()
const ebayProducts = ref()
const aliProducts = ref()
const isLoading = ref(false)

// FUNCTIONS
async function getAmazon() {
    await $fetch(amazonUrl + new URLSearchParams({
        keyword: searchParams.value,
        country: "AU",
      }),
      {
        method: "GET",
        headers: {
            "X-RapidAPI-Host": "amazon24.p.rapidapi.com",
            "X-RapidAPI-Key": rapidKey,
          }
      }).then((res) => amazonProducts.value = res.docs )
}

async function getEbay() {
    await $fetch(ebayUrl + new URLSearchParams({
        page_number: "1",
        product_name: searchParams.value,
        country: "australia",
      }),
      {
        method: "GET",
        headers: {
            "X-RapidAPI-Host": "ebay-data-scraper.p.rapidapi.com",
            "X-RapidAPI-Key": rapidKey,
          }
      }).then((res) => ebayProducts.value = res )
}

async function getAli() {
    await $fetch(aliUrl + new URLSearchParams({
        name: searchParams.value,
        shipToCountry: "AU",
        sort: 'SALE_PRICE_ASC',
      }),
      {
        method: "GET",
        headers: {
            "X-RapidAPI-Host": "magic-aliexpress1.p.rapidapi.com",
            "X-RapidAPI-Key": rapidKey,
          }
      }).then((res) => {
        aliProducts.value = res.docs
        isLoading.value = !isLoading.value
        })
}

const getItems = () => {
    isLoading.value = !isLoading.value
    getAmazon()
    getEbay()
    getAli()
}


</script>

<template>
    <div class="p-6">
        <div class="flex mx-auto space-x-4 mb-4 max-w-screen-md">
            <input type="text" placeholder="Search Part No./Name/SKU etc" v-model="searchParams" @keyup.enter="getItems" class="dark:text-black py-1 px-2 w-full rounded">
            <button class="bg-red-800 px-4 py-2 rounded" @click="getItems"><span v-if="!isLoading">Search</span><span v-if="isLoading">Loading..</span></button>
        </div>

        <div class="flex space-x-4">


            <div class="flex flex-col w-1/3 h-[78vh] overflow-scroll no-bars">
                <h2 class="text-lg p-4 font-bold" v-if="amazonProducts">
                    Amazon.com.au
                </h2>
                <div class="shadow-lg rounded p-4 bg-neutral-700 mb-2 hover:bg-neutral-600" v-for="product in amazonProducts">
                    <a :href="product.product_detail_url" target="_blank" class="flex">
                        <img :src="product.product_main_image_url" :alt="product.product_title" width="100" class="object-contain mr-4 max-h-24">
                        <div>
                            <h3 class="font-semibold">
                                {{ product.product_title.slice(0,70) + '...' }}
                            </h3>
                            <p class="text-red-800 font-semibold text-md">
                                {{ '$' + product.app_sale_price }}
                            </p>
                        </div>
                    </a>
                </div>
            </div>

            <div class="flex flex-col w-1/3 h-[78vh] overflow-scroll no-bars">
                <h2 class="text-lg p-4 font-bold" v-if="ebayProducts">
                    eBay.com.au
                </h2>
                <div class="shadow-lg rounded p-4 bg-neutral-700 mb-2 hover:bg-neutral-600" v-for="product in ebayProducts">
                    <a :href="product.link" target="_blank" class="flex">
                        <img :src="product.thumbnail" :alt="product.description" width="100" class="object-contain mr-4 max-h-24">
                        <div>
                            <h3 class="font-semibold">
                                {{ product.name || product.description || 'Empty Title Error' }}
                            </h3>
                            <p class="text-red-800 font-semibold text-md">
                                {{ product.price }}
                            </p>
                        </div>
                    </a>
                </div>
            </div>
            
            <div class="flex flex-col w-1/3 h-[78vh] overflow-scroll no-bars">
                <h2 class="text-lg p-4 font-bold" v-if="aliProducts">
                    AliExpress.com.au
                </h2>
                <div class="shadow-lg rounded p-4 bg-neutral-700 mb-2 hover:bg-neutral-600" v-for="product in aliProducts">
                    <a :href="product.metadata.image.imageUrl" target="_blank" class="flex">
                        <img :src="product.product_main_image_url" :alt="product.product_title" width="100" class="object-contain mr-4 max-h-24">
                        <div>
                            <h3 class="font-semibold">
                                {{ product.product_title.slice(0,70) + '...' }}
                            </h3>
                            <p class="text-red-800 font-semibold text-md">
                                {{ ' $' + (product.app_sale_price*1.61).toFixed(2) }}
                            </p>
                        </div>
                    </a>
                </div>
            </div>

            

        </div>

    </div>
</template>

<style scoped>
.no-bars::-webkit-scrollbar {
  display: none;
}

.no-bars {
    -ms-overflow-style: none;
    scrollbar-width: none;
}
</style>