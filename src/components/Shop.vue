<template>
    <div class="shop">

        <div class="banner">
            <h2>Online Satış</h2>
            <p>İstediğiniz saatleri en uygun fiyatlarla keşfedin. Şimdi alışveriş yapın ve tarzınızı tamamlayın!</p>
        </div>

        <div class="options">
            <div class="filter">
                <button class="filter-btn" @click="toggleFilter">Filtrele <i class="bi bi-funnel"></i></button>

                <div class="filter-modal" v-if="isFilterVisible">
                    <div class="filter-content">
                        <h3>Ürünleri Filtrele</h3>
                        <div class="hr"></div>
                        
                        <div class="filter-group">
                            <h4>Marka</h4>

                            <div class="inputs">
                                <div class="input-box" v-for="(brand, index) in distinctBrands" :key="index">
                                    <input type="checkbox" :value="brand"> <span>{{ brand }}</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="filter-group">
                            <h4>Kordon Rengi</h4>

                            <div class="inputs">
                                <div class="input-box" v-for="(band, index) in distinctBands" :key="index">
                                    <input type="checkbox" :value="band"> <span>{{ band }}</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="filter-group">
                            <h4>Kasa Rengi</h4>

                            <div class="inputs">
                                <div class="input-box" v-for="(caseColor, index) in distinctCases" :key="index">
                                    <input type="checkbox" :value="caseColor"> <span>{{ caseColor }}</span>
                                </div>
                            </div>
                        </div>

                        <div class="buttons">
                            <button>Filtrele</button>
                            <button @click="toggleFilter">Kapat</button>
                        </div>

                    </div>
                </div>
                <div class="filter-backdrop" v-if="isFilterVisible" @click="toggleFilter"></div>
            </div>

            <div class="sort">
                <button class="sort-btn" @click="toggleSort">Sırala <i class="bi bi-filter"></i></button>

                <div class="sort-modal" v-if="isSortVisible">
                    <div class="sort-content">
                        <h3>Ürünleri Sırala</h3>
                        <div class="hr"></div>

                        <div class="sort-group">
                            <h4>Fiyat</h4>

                            <div class="inputs">
                                <div class="input-box">
                                    <input type="radio" value="Yüksekten düşüğe" name="sorts.price"> <span>Yüksekten düşüğe</span>
                                </div>

                                <div class="input-box">
                                    <input type="radio" value="Düşükten yükseğe" name="sorts.price"> <span>Düşükten yükseğe</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="buttons">
                            <button>Sırala</button>
                            <button @click="toggleSort">Kapat</button>
                        </div>
                    </div>
                </div>
                <div class="sort-backdrop" v-if="isSortVisible" @click="toggleSort"></div>
            </div>
        </div>

        <div class="products">
            <div class="product-card" v-for="(product, index) in products" :key="index">
                <img :src="product.productImages[0].path">
                <div class="description">
                    <h3>{{product.brand}}</h3>
                    <hr>
                    <h4>{{product.model}}</h4>
                    <h5>{{parseFloat(product.price).toFixed(2)}} TL</h5>
                    <button @click="goToProduct(product.id)">Ürünü İncele</button>
                </div>
            </div>
        </div>

        <div class="pagination">
            <button class="prev" :disabled="currentPage === 1" @click="changePage(currentPage - 1)"><i class="bi bi-chevron-left"></i></button>
            <button v-for="page in totalPages" :key="page" :class="['page-btn', {active: page === currentPage}]" @click="changePage(page)">{{page}}</button>
            <!-- <button class="page-btn active">2</button>
            <button class="page-btn">3</button> -->
            <button class="next" :disabled="currentPage === totalPages" @click="changePage(currentPage + 1)"><i class="bi bi-chevron-right"></i></button>
        </div>

    </div>
</template>

<script>
    import axios from 'axios';
    
    export default {
        data(){
            return {
                products: [],
                currentPage: 1,
                pageSize: 9,
                totalPages: 1,
                isFilterVisible: false,
                isSortVisible: false,
                distinctBrands: [],
                distinctBands: [],
                distinctCases: [],
                filterProducts: []
            }
        },
        methods: {
            toggleFilter(){
                this.isFilterVisible = !this.isFilterVisible;
            },
            toggleSort(){
                this.isSortVisible = !this.isSortVisible;
            },
            goToProduct(id){
                this.$router.push(`/product/${id}`)
            },
            async getProducts(){
                let firstResponse = await axios.get('http://18.196.156.3:8080/api/product/get-product-list', {
                    params: {
                        page: 1,
                        pageSize: 36
                    }
                })

                this.filterProducts = firstResponse.data.data;

                this.totalPages = Math.ceil(firstResponse.data.data.length / this.pageSize)

                let secondResponse = await axios.get('http://18.196.156.3:8080/api/product/get-product-list', {
                    params: {
                        page: this.currentPage,
                        pageSize: this.pageSize
                    }
                })

                this.products = secondResponse.data.data;

                console.log(this.products);
                console.log(this.totalPages);

                this.getDistinctBrands();
                this.getDistinctBands();
                this.getDistinctCases();
            },
            getDistinctBrands() {
                // Benzersiz markaları al
                const distinctBrands = [...new Set(this.filterProducts.map(product => product.brand))];
                // console.log("Distinct Brands:", distinctBrands);

                // İsterseniz bu değerleri bir değişkene atayıp filtrelerde gösterebilirsiniz.
                this.distinctBrands = distinctBrands;
            },
            getDistinctBands() {
                // Benzersiz markaları al
                const distinctBands = [...new Set(this.filterProducts.map(product => product.bandColor))];
                // console.log("Distinct Brands:", distinctBrands);

                // İsterseniz bu değerleri bir değişkene atayıp filtrelerde gösterebilirsiniz.
                this.distinctBands = distinctBands;
            },
            getDistinctCases() {
                // Benzersiz markaları al
                const distinctCases = [...new Set(this.filterProducts.map(product => product.caseColor))];
                // console.log("Distinct Brands:", distinctBrands);

                // İsterseniz bu değerleri bir değişkene atayıp filtrelerde gösterebilirsiniz.
                this.distinctCases = distinctCases;
            },
            changePage(newPage) {
                if (newPage >= 1 && newPage <= this.totalPages) {
                    this.currentPage = newPage;
                    this.getProducts();
                }
            },
        },
        mounted() {
            this.getProducts();
        }
    }
</script>

<style scoped>

    .shop{
        width: 100%;
        background-color: whitesmoke;
    }

    /* Banner */

    .shop > .banner{
        width: 100%;
        height: 140px;
        background-color: lightgray;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .shop > .banner > p{
        font-size: 18px;
        font-style: italic;
    }

    /* Options */

    .shop > .options{
        width: 100%;
        height: 60px;
        padding: 0 280px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        background-color: antiquewhite;
    }

    .shop > .options > .filter > .filter-btn{
        width: 140px;
        height: 30px;
        background-color: black;
        color: white;
        border: none;
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        font-size: 16px;
    }

    .shop > .options > .filter > .filter-btn > i{
        margin-top: 2px;
    }

    .shop > .options > .filter > .filter-btn:hover{
        background-color: white;
        color: black;
        border: 1px solid black;
    }

    .shop > .options > .sort > .sort-btn{
        width: 140px;
        height: 30px;
        background-color: black;
        color: white;
        border: none;
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        font-size: 16px;
    }

    .shop > .options > .sort > .sort-btn > i{
        margin-top: 2px;
    }

    .shop > .options > .sort > .sort-btn:hover{
        background-color: white;
        color: black;
        border: 1px solid black;
    }

    /* Products */

    .shop > .products{
        width: 100%;
        padding: 30px 80px;
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(340px, 1fr));
        justify-items: center;
        gap: 60px 30px;
        /* background-color: aquamarine; */
    }

    .shop > .products > .product-card{
        width: 340px;
        height: 400px;
        border: 1px solid black;
    }

    .shop > .products > .product-card > img{
        width: 100%;
        height: 60%;
        display: block;
        object-fit: cover;
    }

    .shop > .products > .product-card > .description{
        width: 100%;
        height: 40%;
    }

    .shop > .products > .product-card > .description > h3{
        margin: 0;
        padding: 5px 0;
        text-align: center;
        font-weight: 500;
        font-size: 22px;
    }

    .shop > .products > .product-card > .description > hr{
        margin: 0;
    }

    .shop > .products > .product-card > .description > h4{
        margin: 5px 0 10px 0;
        text-align: start;
        padding-left: 10px;
        font-weight: 400;
        font-size: 18px;
    }

    .shop > .products > .product-card > .description > h5{
        margin: 0 0 10px 0;
        text-align: center;
        font-weight: 500;
        font-size: 20px;
        font-style: italic;
    }

    .shop > .products > .product-card > .description > button{
        display: flex;
        justify-self: center;
        justify-content: center;
        align-items: center;
        background-color: #DE6449;
        color: white;
        border: 1px solid white;
        width: 128px;
        height: 36px;
        border-radius: 2px;
        transition: 0.2s;
        font-size: 18px;
    }

    .shop > .products > .product-card > .description > button:hover{
        background-color: white;
        color: #DE6449;
        border: 1px solid #DE6449;
    }

    /* Pagination */

    .shop > .pagination{
        width: 100%;
        height: 120px;
        padding-bottom: 30px;
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        /* background-color: blue; */
    }

    .shop > .pagination > button{
        width: 36px;
        height: 36px;
        font-size: 18px;
        border: none;
        background-color: #DE6449;
        color: white;
        transition: 0.2s;
    }

    .shop > .pagination > button.active{
        border: 3px solid black;
    }

    .shop > .pagination > button:hover{
        border: 1px solid #DE6449;
        background-color: white;
        color: #DE6449;
    }

    /* Filter Modal */

    .filter-backdrop {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background-color: rgba(0, 0, 0, 0.5); /* Shadow effect */
        z-index: 1000;
    }

    .filter-modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 500px;
        height: 100vh;
        background-color: white;
        z-index: 1100;
        overflow-y: scroll;
        box-shadow: 2px 0 5px rgba(0, 0, 0, 0.3);
    }

    .filter-modal .buttons{
        width: 100%;
        padding: 10px 0;
        display: flex;
        justify-content: center;
        align-items: center;
        column-gap: 40px;
        /* background-color: #DE6449; */
    }

    .filter-modal .buttons button {
        width: 180px;
        height: 40px;
        font-size: 18px;
        background-color: black;
        color: white;
        border: none;
        cursor: pointer;
        transition: 0.2s;
    }

    .filter-modal .buttons button:nth-of-type(2) {
        background-color: red;
    }

    .filter-modal .buttons button:hover {
        background-color: white;
        color: black;
        border: 1px solid black;
    }

    .filter-modal .buttons button:nth-of-type(2):hover {
        background-color: white;
        color: red;
        border: 1px solid red;
    }

    .filter-content {
        padding: 20px 0;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 2px;
        /* background-color: aquamarine; */
    }

    .filter-content h3{
        margin: 0 0 5px 0;
    }

    .filter-content .hr{
        width: 100%;
        margin: 0;
        padding: 0;
        height: 1px;
        background-color: black;
    }

    .filter-group{
        width: 100%;
        padding: 10px 20px;
        /* background-color: blueviolet; */
    }

    .filter-group h4{
        /* background-color: yellow; */
        margin: 0 0 10px 0;
    }

    .filter-group .inputs{
        width: 100%;
        padding: 5px 10px;
        display: flex;
        flex-wrap: wrap;
        column-gap: 20px;
        row-gap: 14px;
        /* background-color: yellowgreen; */
    }

    .filter-group .inputs .input-box{
        display: flex;
        justify-content: center;
        align-items: center;
        column-gap: 6px;
        font-size: 18px;
        /* background-color: tomato; */
    }

    .filter-group .inputs .input-box input{
        width: 18px;
        height: 18px;
        cursor: pointer;
    }

    /* Sort Modal */

    .sort-backdrop {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background-color: rgba(0, 0, 0, 0.5); /* Shadow effect */
        z-index: 1000;
    }

    .sort-modal {
        position: fixed;
        top: 0;
        right: 0;
        width: 500px;
        height: 100vh;
        background-color: white;
        z-index: 1100;
        box-shadow: 2px 0 5px rgba(0, 0, 0, 0.3);
    }

    .sort-content {
        padding: 20px 0;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 2px;
        /* background-color: aquamarine; */
    }

    .sort-modal .buttons{
        width: 100%;
        padding: 10px 0;
        display: flex;
        justify-content: center;
        align-items: center;
        column-gap: 40px;
        /* background-color: #DE6449; */
    }

    .sort-modal .buttons button {
        width: 180px;
        height: 40px;
        font-size: 18px;
        background-color: black;
        color: white;
        border: none;
        cursor: pointer;
        transition: 0.2s;
    }

    .sort-modal .buttons button:nth-of-type(2) {
        background-color: red;
    }

    .sort-modal .buttons button:hover {
        background-color: white;
        color: black;
        border: 1px solid black;
    }

    .sort-modal .buttons button:nth-of-type(2):hover {
        background-color: white;
        color: red;
        border: 1px solid red;
    }

    .sort-content h3{
        margin: 0 0 5px 0;
    }

    .sort-content .hr{
        width: 100%;
        margin: 0;
        padding: 0;
        height: 1px;
        background-color: black;
    }

    .sort-group{
        width: 100%;
        padding: 10px 20px;
        /* background-color: blueviolet; */
    }

    .sort-group h4{
        /* background-color: yellow; */
        margin: 0 0 10px 0;
    }

    .sort-group .inputs{
        width: 100%;
        padding: 5px 10px;
        display: flex;
        flex-wrap: wrap;
        column-gap: 20px;
        row-gap: 14px;
        /* background-color: yellowgreen; */
    }

    .sort-group .inputs .input-box{
        display: flex;
        justify-content: center;
        align-items: center;
        column-gap: 6px;
        font-size: 18px;
        /* background-color: tomato; */
    }

    .sort-group .inputs .input-box input{
        width: 18px;
        height: 18px;
        cursor: pointer;
    }

</style>