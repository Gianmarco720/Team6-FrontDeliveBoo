<script>
import { state } from '../state';
import axios from 'axios'

var modal = document.getElementById("myModal");
window.onclick = function (event) {
    if (event.target == modal) {
        modal.style.display = "none";
    }
}

export default {
    name: 'RestaurantView',
    props: ['slug'],
    data() {
        return {
            state,
            loading: true,
            restaurant: [],
            has_dishes: true,
            new_dish_cart: [],
            //oggetti del carrello

        }
    },
    methods: {
        getSingleRestaurant(url) {
            // console.log(url);
            axios
                .get(url)
                .then(response => {


                    this.loading = true
                    this.restaurant = response.data.data;
                    this.getHasDishes()
                    // console.log(this.restaurant);
                    this.loading = false
                    this.state.single_restaurant = this.restaurant
                })
                .catch(error => {
                    console.error(error.message);
                })
                .finally(function () {

                })
        },
        cancelModal() {
            //in questo caso tolgo la modale e non ci metto il piatto che si voleva aggiungere

            this.state.new_dish_cart = []
            this.state.cart_counter = this.state.cart.length
            //ritorna il carrello di prima
            // for (let i = 0; i < this.state.cart.length; i++) {

            //     this.state.total_cart += parseFloat(this.state.cart[i].price)

            // }
            // this.state.total_cart = 
            var modal = document.getElementById("myModal");
            modal.style.display = "none";

        },
        acceptModal() {
            //in questo caso tolgo la modale e cancello il carrello e ci metto il piatto che si voleva aggiungere

            this.state.cart = []
            this.state.checkQtyDish(this.state.new_dish_cart)
            // this.cart.push(this.new_dish_cart)

            this.state.cart_counter = this.state.cart.length
            // localStorage.setItem("cart", JSON.stringify(this.cart))

            this.state.total_cart = this.state.getTotal_cart(this.state.cart)

            localStorage.setItem("total", JSON.stringify(this.state.total_cart))

            var modal = document.getElementById("myModal");
            modal.style.display = "none";
            // state.getTotal_cart()

            //azzero il totale e aggiungo nuovo prezzo

            // const price = parseFloat(this.state.new_dish_cart.price)
            // console.log(this.total_cart, 'totale');


        },
        getHasDishes() {

            //determino se ha piatti o meno
            if (!this.restaurant) {
                this.has_dishes = false
            }
            if (!this.restaurant.dishes) {
                this.has_dishes = false
            } else {
                if (!this.restaurant.dishes.length) {
                    this.has_dishes = false
                }
            }

            //inizializzo il carrello nel local storage
            if (this.has_dishes) {

                localStorage.setItem('dishes', JSON.stringify(this.restaurant.dishes))
                if (!localStorage.getItem("cart")) {
                    localStorage.setItem("cart", "[]")

                    this.state.cart_counter = 0
                }
            }
        },

        getBack() {

            this.$router.go(-1)
        }

    },

    mounted() {

        this.getSingleRestaurant(this.state.baseUrl + 'api/restaurants/' + this.slug)


    }
}
</script>

<template>
    <section class="p-0">
        <!-- <a @click="getBack()">back</a> -->
        <div class="container py-5">

            <!-- The Modal -->
            <div id="myModal" class="modal">

                <!-- Modal content -->
                <div class="modal-content text-center">

                    <h3>You can only order from one restaurant</h3>
                    <p>ok to continue or cancel to go back</p>
                    <div class="d-flex justify-content-center">
                        <button class=" m-3 btn btn-primary " @click="acceptModal()">Ok</button>
                        <button type="button" class=" m-3 btn btn-danger" @click="cancelModal()">Cancel</button>
                    </div>

                </div>

            </div>



            <div class="row flex-md-row flex-column ">

                <!-- banner con la foto del ristorante -->
                <div class="col-lg-3 col-md-6 restaurants mb-3">
                    <div class="my-card-info">
                        <div class="d-flex justify-content-center">
                            <img class="mb-4 cover" :src="state.imagePath(restaurant.cover_image)" alt="">
                        </div>

                        <!-- info del ristorante -->
                        <div class="d-flex justify-content-center text-center">

                            <div class="">
                                <h1 class="">{{ this.restaurant.name }}</h1>
                                <div class="d-flex justify-content-center">
                                    <div v-for="types in this.restaurant.tipologies" class="type">{{ types.name }}</div>
                                </div>
                                <div class="">
                                    <div class="d-flex m-2">
                                        <font-awesome-icon icon="fa-solid fa-location-dot" />
                                        <div class="px-3">{{ this.restaurant.address }}</div>

                                    </div>
                                    <div class="d-flex m-2">
                                        <font-awesome-icon icon="fa-solid fa-phone" />
                                        <div class="px-3"> {{ this.restaurant.phone_number }}</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- cart sm-->
                <div class="d-lg-none col">
                    <div class="my-card-info">
                        <div class="col-4 w-100">
                            <div class="d-flex justify-content-between align-items-center mb-4">
                                <div class="fs-2 fw-bold">Cart</div>
                                <div v-if="state.total_cart == null" class="main-color fs-3 fw-bold">Total price: 0,00 ???
                                </div>
                                <div v-else class="main-color fs-3 fw-bold">Total price: {{ state.total_cart.toFixed(2) +
                                    '???' }}
                                </div>
                            </div>
                            <div class="cart-h">
                                <div v-for="dish in state.cart">

                                    <hr>
                                    <div class="d-flex justify-content-around align-items-center">
                                        <div class="my-3">
                                            <h3 class="pr-5">{{ dish.name }}</h3>
                                            <div class="d-flex align-items-center justify-content-between">
                                                <div class="me-3 main-color">{{ dish.price + '???' }}</div>
                                                <div>Qty:<span> {{ dish.qty }}</span> </div>
                                            </div>
                                        </div>

                                        <div class="btn-wrapper d-flex flex-column justify-content-center">
                                            <button @click="state.addDishToCart(dish)"
                                                class="my-4 mx-3 btn btn-sm btn-primary rounded-pill px-3 fs-4">
                                                Add
                                            </button>
                                            <button type="button" @click="state.removeDishToCart(dish)"
                                                class=" my-4 mx-3 btn btn-sm btn-danger rounded-pill px-3 fs-4">
                                                Remove
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>
                    </div>

                </div>

                <!-- piatti -->
                <div class="col-lg-6 col-12 d-flex justify-content-center dishes mt-lg-0 mt-5">
                    <div class="col-10" v-if="this.has_dishes">
                        <div v-for="dish in this.restaurant.dishes">
                            <div class="my-card d-flex p-1 my-3">
                                <div class="d-sm-block d-none col-5">
                                    <img :src="state.imagePath(dish.cover_image)" alt="">
                                </div>
                                <div class="details p-3 flex-grow-1">
                                    <h2>{{ dish.name }}</h2>
                                    <p>{{ dish.description }}</p>
                                    <h4 class="pr-3"> {{ dish.price }} <span>&#8364;</span></h4>
                                </div>
                                <div class="align-self-center text-end">
                                    <button @click="state.addDishToCart(dish)"
                                        class="btn py-2 px-3 mx-4 btn-primary btn-sm d-flex align-items-center">
                                        Add
                                    </button>
                                </div>

                            </div>

                        </div>
                    </div>
                    <h2 class=" py-5 text-center" v-else>There are no dishes for this restaurant yet</h2>

                </div>

                <button @click="state.router.push({ name: 'checkout' })"
                    class=" only-sm my-5 w-100 my-btn">Checkout</button>

                <!-- cart lg-->
                <div class="col-3 d-lg-block d-none">
                    <div class="my-card-info">
                        <div class="col-4 w-100">
                            <div class="d-flex justify-content-between align-items-center mb-4 flex-xl-row flex-column">
                                <div class="fs-2 fw-bold">Cart</div>
                                <div v-if="state.total_cart == null" class="main-color fs-3 fw-bold">Total price: 0,00 ???
                                </div>
                                <div v-else class="main-color fs-3 fw-bold">Total price: {{ state.total_cart.toFixed(2) +
                                    '???' }}
                                </div>
                            </div>

                            <div class="cart-h">
                                <div v-for="dish in state.cart">

                                    <hr>
                                    <div class="d-flex justify-content-around align-items-center">
                                        <div class="my-3">
                                            <h3 class="pr-5">{{ dish.name }}</h3>
                                            <div class="d-flex align-items-center justify-content-between">
                                                <div class="me-3 main-color">{{ dish.price + '???' }}</div>
                                                <div>Qty:<span> {{ dish.qty }}</span> </div>
                                            </div>
                                        </div>

                                        <div class="btn-wrapper d-flex flex-column justify-content-center">
                                            <button @click="state.addDishToCart(dish)"
                                                class="my-4 mx-3 btn btn-sm btn-primary rounded-pill px-3 fs-4">
                                                Add
                                            </button>
                                            <button type="button" @click="state.removeDishToCart(dish)"
                                                class=" my-4 mx-3 btn btn-sm btn-danger rounded-pill px-3 fs-4">
                                                Remove
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>
                    </div>

                    <button @click="state.router.push({ name: 'checkout' })" class=" my-5 w-100 my-btn">Checkout</button>
                </div>
            </div>


        </div>
    </section>
</template>


<style lang="scss" scoped>
@import '../styles/general.scss';

.my-btn {
    border: 1px solid transparent;

    &:hover {
        color: var(--main-color);
        border-color: var(--main-color);
    }
}

.only-sm {
    display: none;
}

@media (max-width: 992px) {
    .only-sm {
        display: block;

    }
}



.cart-h {

    overflow-y: overlay;
    max-height: 330px;


}

:root {

    --main-color: #e29436;
    --main-color-dark: #cf4835;

}

.dishes {
    img {
        width: 100%;
    }
}


.restaurants {
    img {
        border-radius: 5px;
        object-fit: cover;
        object-position: center;
        height: 20rem;
    }
}


.modal {
    display: none;
    /* Hidden by default */
    position: fixed;
    /* Stay in place */
    z-index: 1;
    /* Sit on top */
    padding-top: 100px;
    /* Location of the box */
    left: 0;
    top: 0;
    width: 100%;
    /* Full width */
    height: 100%;
    /* Full height */
    overflow: auto;
    /* Enable scroll if needed */
    background-color: rgb(0, 0, 0);
    /* Fallback color */
    background-color: rgba(0, 0, 0, 0.4);
    /* Black w/ opacity */
}

/* Modal Content */
.modal-content {
    background-color: #fefefe;
    margin: auto;
    padding: 20px;
    border: 1px solid #888;
    width: 80%;
}

.cover {
    max-height: 300px;
    width: 100%;
}




.type {
    padding: 5px;
}
</style>