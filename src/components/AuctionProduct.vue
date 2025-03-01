<template>
    <div class="auction-product">

        <div class="product-card" v-if="product && product.productDto">
            <div class="product-details">
                <Carousel v-bind="carouselConfig">
                    <Slide v-for="(image, index) in product.productDto.productImages" :key="index">
                        <div class="carousel__item">
                            <img :src="image.path">
                        </div>
                    </Slide>

                    <template #addons>
                        <Navigation />
                        <Pagination />
                    </template>
                </Carousel>

                <div class="description">
                    <h3>{{product.productDto.brand}}</h3>
                    <h4>{{product.productDto.model}}</h4>
                    <h5>Teknoloji: <span>{{product.productDto.technology}}</span></h5>
                    <h5>Kadran Rengi: <span>{{product.productDto.dialColor}}</span></h5>
                    <h5>Kasa Tipi: <span>{{product.productDto.caseShape}}</span></h5>
                    <h5>Kasa Rengi: <span>{{product.productDto.caseColor}}</span></h5>
                    <h5>Kordon Rengi: <span>{{product.productDto.bandColor}}</span></h5>
                    <h5>Cinsiyet: <span>{{product.productDto.gender}}</span></h5>
                </div>
            </div>

            <div class="auction-details">
                <h2>Müzayede Bilgileri</h2>
                <div class="hr"></div>

                <div class="description">
                    <h3>Başlangıç Zamanı: {{formatDate(product.startDate)}}</h3>
                    <h3>Bitiş Zamanı: {{formatDate(product.endDate)}}</h3>
                </div>

                <div class="countdown">
                    <h3 v-if="isActive" class="continue"><i class="bi bi-broadcast"></i> Müzayede Devam Ediyor</h3>
                    <h3 v-else class="end"><i class="bi bi-x-circle"></i> Müzayede Aktif Değil</h3>
                </div>

                <div class="bids">
                    <h3>Başlangıç Fiyatı: {{(product.startingPrice).toFixed(2)}} TL</h3>
                    <h3>Son Teklif: {{(product.currentPrice).toFixed(2)}} TL</h3>
                </div>

                <input placeholder="Teklifinizi girin..." type="number" v-model="bid">
                <button @click="bidToProduct">Teklif Ver</button>
            </div>
        </div>
        <div v-else>Loading...</div>
    </div>
</template>

<script>
import 'vue3-carousel/carousel.css'
import { Carousel, Slide, Navigation, Pagination } from 'vue3-carousel';
import axios from 'axios';
import { jwtDecode } from 'jwt-decode';

export default{
    components: {
        Carousel,
        Slide,
        Pagination,
        Navigation
    },
    data(){
        return{
            carouselConfig: {
                wrapAround: true
            },
            product: { },
            auctionId: this.$route.params.id,
            isActive: false,
            bid: null,
            connection: null, // SignalR bağlantısı
        }
    },
    methods: {
        async bidToProduct(){
            try {
                let userId = jwtDecode(this.$store.state.token).Id;

                if(this.bid === null || this.bid === ""){
                    alert("Bir teklif girmeniz gerek!")
                    return
                }
                if(this.bid.toString().includes(",") || this.bid.toString().includes(".")){
                    alert("Teklifte virgül veya nokta gibi işaretler bulunmamalı!")
                    return
                }
                if(this.bid <= this.product.startingPrice || this.bid <= this.product.currentPrice){
                    alert("Teklifiniz başlangıç fiyatı ve son tekliften büyük olmalı!");
                    return;
                }
                if(userId === this.product.userId){
                    alert("Kendi müzayede ürününüze teklif yapamazsınız!");
                    return;
                }

                let reqBody = {
                    userId: userId,
                    auctionId: this.auctionId,
                    bidAmount: this.bid 
                }

                let response = await axios.post('http://18.196.156.3:8080/api/auction/bid-on-auction', reqBody, {
                    headers: {
                        Authorization: `Bearer ${this.$store.state.token}`
                    }
                });

                console.log(response);

                alert("Teklifiniz kabul edildi!");
            } catch (error) {
                console.error(error);
                alert("Bir hata oluştu.");
            }

        },
        formatDate(dateString) {
            const date = new Date(dateString);
            date.setHours(date.getHours() + 3);
            const year = date.getUTCFullYear();
            const month = String(date.getUTCMonth() + 1).padStart(2, "0");
            const day = String(date.getUTCDate()).padStart(2, "0");
            const hours = String(date.getUTCHours()).padStart(2, "0");
            const minutes = String(date.getUTCMinutes()).padStart(2, "0");

            return `${year}-${month}-${day} ${hours}:${minutes}`;
        },
        async getAuctionProduct(){
            try {

                let response = await axios.get('http://18.196.156.3:8080/api/auction/get-all-active-auctions');

                console.log(response.data.data);

                let responseTwo = await axios.get('http://18.196.156.3:8080/api/auction/get-auction-by-id', {
                    headers: {
                        Authorization: `Bearer ${this.$store.state.token}`,
                        actionId: this.auctionId
                    }
                })

                if(response.data.data.find((item) => item.auctionId === this.auctionId)){
                    this.isActive = true;
                }

                console.log(this.isActive);

                console.log(responseTwo.data.data);

                this.product = responseTwo.data.data;

            } catch (error) {
                console.error(error);
            }
        },
        startSignalRConnection() {
            this.connection = new signalR.HubConnectionBuilder()
                .withUrl("http://18.196.156.3:8080/auctionHub")
                .withAutomaticReconnect()
                .build();

            this.connection
                .start()
                .then(() => {
                console.log("SignalR bağlantısı kuruldu.");

                // Son teklif güncellemelerini dinle
                    this.connection.on("ReceiveLastBidAtAuction", (data) => {
                        console.log("Son teklif güncellendi:", data);
                        if(data.auctionId === this.auctionId){
                            this.product.currentPrice = data.bidAmount;
                        }
                    });
                })
                .catch((err) => console.error("SignalR bağlantı hatası:", err));
            },
    },
    mounted() {
        this.getAuctionProduct();
        this.startSignalRConnection(); // SignalR bağlantısını başlat
    },
    beforeMount(){
        if(!this.$store.state.token){
            this.$router.push('/login');
        }
    },
    beforeDestroy() {
        if (this.connection) {
        this.connection.stop(); // Bileşen yok edilmeden önce bağlantıyı durdur
        }
    }
}

</script>

<style scoped>
.auction-product{
    max-width: 100%;
    padding: 40px 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: whitesmoke;
}

.auction-product > .product-card{
    display: flex;
    box-shadow: 0 0 8px rgba(0, 0, 0, 0.5);
}

/* Product Details */

.auction-product > .product-card > .product-details{
    width: 440px;
    height: 620px;
    display: flex;
    flex-direction: column;
    border-right: 1px solid black;
}

/* .auction-product > .product-card > .product-details > img{
    width: 100%;
    height: 60%;
    object-fit: contain;
    background-color: #ffffff;
} */



.carousel{
    width: 100%;
    height: 372px;
    background-color: #ffffff;
}

.carousel__slide{
    width: 90%;
    height: 372px;
    /* background-color: red; */
}

.carousel__item{
    width: 90%;
    height: 372px;
}

.carousel__item img{
    width: 90%;
    height: 372px;
}

:deep(.carousel){
    --vc-nav-width: 60px;
    --vc-nav-height: 60px;
    --vc-nav-color: black;
    --vc-pgn-height: 8px;
    --vc-pgn-width: 20px;
    --vc-pgn-active-color: black;
}

.auction-product > .product-card > .product-details > .description{
    width: 100%;
    height: 40%;
    display: flex;
    flex-direction: column;
    background-color: antiquewhite;
}

.auction-product > .product-card > .product-details > .description > h3{
    padding: 0 0 0 20px;
    margin: 12px 0 0 0;
    font-size: 22px;
    font-weight: 500;
    /* background-color: aqua; */
}

.auction-product > .product-card > .product-details > .description > h4{
    padding: 0 0 0 20px;
    margin: 0 0 10px 0;
    font-size: 18px;
    font-weight: 400;
    /* background-color: aqua; */
}

.auction-product > .product-card > .product-details > .description > h5{
    padding: 0 0 0 20px;
    margin: 0 0 3px 0;
    font-size: 20px;
    font-weight: 500;
    /* background-color: aqua; */
}

.auction-product > .product-card > .product-details > .description > h5 > span{
    font-style: italic;
    /* background-color: aqua; */
}


/* Auction Details */

.auction-product > .product-card > .auction-details{
    width: 500px;
    height: 620px;
    display: flex;
    flex-direction: column;
    /* gap: 20px; */
    background-color: #ffffff;
}

.auction-product > .product-card > .auction-details > h2{
    padding: 15px 0 15px 0;
    margin: 0;
    font-weight: 400;
    text-align: center;
    color: black;
    /* background-color: antiquewhite; */
}

.auction-product > .product-card > .auction-details > .hr{
    width: 100%;
    height: 2px;
    padding: 0;
    margin: 0;
    background-color: black;
}

.auction-product > .product-card > .auction-details > .description{
    width: 100%;
    display: flex;
    flex-direction: column;
    padding: 20px 0 20px 10px;
    gap: 10px;
    /* background-color: antiquewhite; */
}

.auction-product > .product-card > .auction-details > .description > h3{
    padding: 0;
    margin: 0;
    text-align: center;
    font-size: 22px;
    font-weight: 400;
    color: black;
}

.auction-product > .product-card > .auction-details > .countdown{
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 50px 0;
    /* background-color: #DE6449; */
}

.auction-product > .product-card > .auction-details > .countdown > h3.continue{
    padding: 0;
    margin: 0;
    font-size: 22px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 400;
    color: green;
}

.auction-product > .product-card > .auction-details > .countdown > h3.end{
    padding: 0;
    margin: 0;
    font-size: 22px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 400;
    color: gray;
}

.auction-product > .product-card > .auction-details > .bids{
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    row-gap: 10px;
    padding: 20px 0;
    /* background-color: #DE6449; */
}

.auction-product > .product-card > .auction-details > .bids > h3{
    padding: 0;
    margin: 0;
    font-size: 26px;
    font-weight: 400;
}

.auction-product > .product-card > .auction-details > .bids > h3:nth-of-type(1){
    color: red;
}

.auction-product > .product-card > .auction-details > .bids > h3:nth-of-type(2){
    color: green;
}

.auction-product > .product-card > .auction-details > input{
    width: 220px;
    height: 40px;
    margin-top: 30px;
    font-size: 20px;
    font-weight: 400;
    border: 1px solid black;
    color: black;
    display: block;
    align-self: center;
    text-align: center;
}

.auction-product > .product-card > .auction-details > button{
    width: 160px;
    height: 40px;
    font-size: 22px;
    font-weight: 400;
    margin-top: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
    align-self: center;
    color: white;
    border: none;
    background-color: #DE6449;
    transition: 0.2s;
}

.auction-product > .product-card > .auction-details > button:hover{
    color: #DE6449;
    border: 1px solid #DE6449;
    background-color: white;
}

/* Testimonials */

</style>