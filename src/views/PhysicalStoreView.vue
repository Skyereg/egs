<template>
    <div class="category_wrapper">
        <div class="page-head">
            <div class="container">
                Home <i class="fa-solid fa-chevron-right"></i> {{ this.$route.meta.category_name }} <i class="fa-solid fa-chevron-right"></i> {{ this.$route.params.name.replace(/-/g, " ").charAt(0).toUpperCase() + this.$route.params.name.replace(/-/g, " ").slice(1) }}
            </div>
        </div>
        <div class="container sub_categories" v-if="subCategories && subCategories.length > 0">
            <a :href="`/physical-store/${item.name.toLowerCase().replace(/\s+/g, '-')}/${item.id}`" v-for="item in subCategories" :key="item.id">
                <div class="img">
                    <img :src="item.logo" :alt="item.name">
                </div>
                <h4>
                    {{ item.name }}
                </h4>
            </a>
        </div>
        <div class="container products" v-if="products && products.length > 0">
            <div class="head">
                <h2>All Products</h2>
                <div class="sort">
                    <div>
                        Showing <span>{{ page }} - {{ per_page }}</span> of {{ total }} Results
                    </div>
                    <div>
                        <select name="per_pag" id="per_page" v-model="per_page" @change="this.page = 1; fetchProducts(categoryId)">
                            <option value="20" selected>Show 20 items</option>
                            <option value="40">Show 40 items</option>
                            <option value="60">Show 60 items</option>
                        </select>
                    </div>
                    <div>
                        Sort By: 
                        <select name="per_pag" id="per_page">
                            <option value="" selected>.................</option>
                            <option value="1">Price Low To High</option>
                            <option value="2">Price Hight To Low</option>
                        </select>
                    </div>
                </div>
            </div>
            <hr>
            <div class="body">
                <div class="product" v-for="item in products" :key="item.id">
                    <img src="./../assets/imgs/shipping_abroad.png" alt="shipping abroad" class="shipping_icon" v-if="item.shipping_abroad">
                    <a :href="`/product/${item.id}`">
                        <div class="img">
                            <img :src="item.main_image" :alt="item.name">
                            <p>{{ item.sub_category.name }}</p>
                            <h4>
                                {{ item.name.length >= 39 ? item.name.slice(0, 39) + '...' :  item.name }}
                            </h4>
                        </div>
                        <div>
                            <div class="rate">
                                <div class="stars">
                                    <i class="fa-regular fa-star active"></i>
                                    <i class="fa-regular fa-star active"></i>
                                    <i class="fa-regular fa-star active"></i>
                                    <i class="fa-regular fa-star active"></i>
                                    <i class="fa-regular fa-star"></i></div>
                                ( 3 Reviews ) 
                            </div>
                            <div class="price">
                                <h1 v-if="item.price_after_discount">{{ item.price_after_discount ? item.price_after_discount.toLocaleString() : '' }}</h1>
                                <h1>{{ item.price.toLocaleString() }}</h1>
                            </div>
                        </div>
                    </a>
                    <button class="add-to-cart" @click="addProductToCart(item.id, 1, item.stock, item.type)">
                        Add To Cart
                    </button>
                    <button :class="item.isFav ? 'active' : ''" class="add-to-wishlist" @click="likeProduct(item.id)">
                        <i class="fa-regular fa-heart"></i> Add To Wishlist
                    </button>
                </div>
            </div>
            <div class="pagination" v-if="last_page > 1">
                <div v-for="page_num in last_page" :key="page_num" >
                    <label :for="`page_num_${page_num}`" :class="page_num == page ? 'active' : ''">{{ page_num }}</label>
                    <input type="radio" name="page_num" :id="`page_num_${page_num}`" v-model="page" :value="page_num" @change="fetchProducts(categoryId)">
                </div>
            </div>
        </div>

        <div class="container not_products" v-if="(!products || !products[0]) && (!subCategories || !subCategories[0])">
            <h1 style="margin: 5rem 0; text-align:center;color: #717171;" v-if="showNotProducts">This Category does not contain any product yet!</h1>
        </div>
    </div>
</template>

<script>
global.jQuery = require('jquery');
var $ = global.jQuery;
window.$ = $;
// import { router } from 'vue';
import axios from 'axios';

export default {
    name: 'PhysicalStoreView',
    data() {
        return {
            categoryData: null,
            products: null,
            categoryType: this.$route.meta.type,
            categoryId: this.$route.params.id,
            subCategories: null,
            per_page: 20,
            page: 1,
            total: 0,
            last_page: 0,
            showNotProducts: false,
            cart: null,
            products_cart: null,
            cards_cart: null,
        }
    },
    methods: {
        async fetchSubCategories(categoryId) {
            $('.loader').fadeIn().css('display', 'flex')
            try {
                const response = await axios.get(`https://api.egyptgamestore.com/api/categories/children/children?category_id=${categoryId}`);
                if (response.data.status === true) {
                    this.subCategories = response.data.data
                    $('.loader').fadeOut()
                    setTimeout(() => {
                        $('#errors').fadeOut('slow')
                    }, 4000);
                } else {
                    $('.loader').fadeOut()
                    // document.getElementById('errors').innerHTML = ''
                    // $.each(response.data.errors, function (key, value) {
                    //     let error = document.createElement('div')
                    //     error.classList = 'error'
                    //     error.innerHTML = value
                    //     document.getElementById('errors').append(error)
                    // });
                    // $('#errors').fadeIn('slow')
                    // setTimeout(() => {
                    //     $('input').css('outline', 'none')
                    //     $('#errors').fadeOut('slow')
                    // }, 3500);
                }

            } catch (error) {
                document.getElementById('errors').innerHTML = ''
                let err = document.createElement('div')
                err.classList = 'error'
                err.innerHTML = 'server error try again later'
                document.getElementById('errors').append(err)
                $('#errors').fadeIn('slow')
                $('.loader').fadeOut()

                setTimeout(() => {
                    $('#errors').fadeOut('slow')
                }, 3500);

                console.error(error);
            }
        },
        async fetchProducts(categoryId) {
            $('.loader').fadeIn().css('display', 'flex')
            try {
                const response = await axios.get(`https://api.egyptgamestore.com/api/products/category?category_id=${categoryId}&per_page=${this.per_page}&page=${this.page}`, {
                    headers: {
                        "AUTHORIZATION": 'Bearer ' + sessionStorage.getItem('user_token')
                    }
                });
                if (response.data.status === true) {
                    this.products = response.data.data.products
                    this.total = response.data.data.total
                    this.last_page = response.data.data.last_page
                    if (!this.products || !this.products[0])
                        this.showNotProducts = true
                    $('.loader').fadeOut()
                    setTimeout(() => {
                        $('#errors').fadeOut('slow')
                    }, 4000);
                } else {
                    $('.loader').fadeOut()
                    document.getElementById('errors').innerHTML = ''
                    $.each(response.data.errors, function (key, value) {
                        let error = document.createElement('div')
                        error.classList = 'error'
                        error.innerHTML = value
                        document.getElementById('errors').append(error)
                    });
                    $('#errors').fadeIn('slow')
                    setTimeout(() => {
                        $('input').css('outline', 'none')
                        $('#errors').fadeOut('slow')
                    }, 3500);
                }

            } catch (error) {
                document.getElementById('errors').innerHTML = ''
                let err = document.createElement('div')
                err.classList = 'error'
                err.innerHTML = 'server error try again later'
                document.getElementById('errors').append(err)
                $('#errors').fadeIn('slow')
                $('.loader').fadeOut()

                setTimeout(() => {
                    $('#errors').fadeOut('slow')
                }, 3500);

                console.error(error);
            }
        },
        async likeProduct(product_id) {
            try {
                const response = await axios.post(`https://api.egyptgamestore.com/api/products/${product_id}/liked`, {
                },
                    {
                        headers: {
                            "AUTHORIZATION": 'Bearer ' + sessionStorage.getItem('user_token')
                        }
                    },
                );
                if (response.data.status === true) {
                    document.getElementById('errors').innerHTML = ''
                    $('.loader').fadeOut()
                    this.fetchProducts(this.categoryId)
                } else {
                    $('.loader').fadeOut()
                    document.getElementById('errors').innerHTML = ''
                    $.each(response.data.errors, function (key, value) {
                        let error = document.createElement('div')
                        error.classList = 'error'
                        error.innerHTML = value
                        document.getElementById('errors').append(error)
                    });
                    $('#errors').fadeIn('slow')
                    
                    setTimeout(() => {
                        $('input').css('outline', 'none')
                        $('#errors').fadeOut('slow')
                    }, 3500);
                }

            } catch (error) {
                document.getElementById('errors').innerHTML = ''
                let err = document.createElement('div')
                err.classList = 'error'
                err.innerHTML = 'server error try again later'
                document.getElementById('errors').append(err)
                $('#errors').fadeIn('slow')
                $('.loader').fadeOut()

                setTimeout(() => {
                    $('#errors').fadeOut('slow')
                }, 3500);

                console.error(error);
            }
        },
        async addProductToCart(product_id, qty, product_valid_qty, product_stock) {
            if (product_stock == 2) {
                document.getElementById('errors').innerHTML = ''
                let error = document.createElement('div')
                error.classList = 'error'
                error.innerHTML = 'This product is not available now'
                document.getElementById('errors').append(error)
                $('#errors').fadeIn('slow')

                setTimeout(() => {
                    $('input').css('outline', 'none')
                    $('#errors').fadeOut('slow')
                }, 3500);

            } else if (product_valid_qty < qty && product_stock == 1) {
                document.getElementById('errors').innerHTML = ''
                let error = document.createElement('div')
                error.classList = 'error'
                error.innerHTML = 'This quantity is not available'
                document.getElementById('errors').append(error)
                $('#errors').fadeIn('slow')

                setTimeout(() => {
                    $('input').css('outline', 'none')
                    $('#errors').fadeOut('slow')
                }, 3500);
            } else {
                $('.loader').fadeIn().css('display', 'flex')
                try {
                    const response = await axios.post(`https://api.egyptgamestore.com/api/products/${product_id}/add-cart`, {
                        qty: qty,
                        type: 'add',
                    },
                        {
                            headers: {
                                "AUTHORIZATION": 'Bearer ' + sessionStorage.getItem('user_token')
                            }
                        },
                    );
                    if (response.data.status === true) {
                        document.getElementById('errors').innerHTML = ''
                        let error = document.createElement('div')
                        error.classList = 'success'
                        error.innerHTML = response.data.message
                        document.getElementById('errors').append(error)
                        $('#errors').fadeIn('slow')
                        setTimeout(() => {
                            $('#errors').fadeOut('slow')
                            $('.loader').fadeOut()
                            if (!this.cart || !this.cart.length) {
                                window.location.reload()
                            }
                        }, 1000);
                    } else {
                        $('.loader').fadeOut()
                        document.getElementById('errors').innerHTML = ''
                        $.each(response.data.errors, function (key, value) {
                            let error = document.createElement('div')
                            error.classList = 'error'
                            error.innerHTML = value
                            document.getElementById('errors').append(error)
                        });
                        $('#errors').fadeIn('slow')

                        setTimeout(() => {
                            $('input').css('outline', 'none')
                            $('#errors').fadeOut('slow')
                        }, 3500);
                    }

                } catch (error) {
                    document.getElementById('errors').innerHTML = ''
                    let err = document.createElement('div')
                    err.classList = 'error'
                    err.innerHTML = 'server error try again later'
                    document.getElementById('errors').append(err)
                    $('#errors').fadeIn('slow')
                    $('.loader').fadeOut()

                    setTimeout(() => {
                        $('#errors').fadeOut('slow')
                    }, 3500);

                    console.error(error);
                }
            }
        },
        async getCart() {
            try {
                const response = await axios.get(`https://api.egyptgamestore.com/api/users/cart`,
                    {
                        headers: {
                            "AUTHORIZATION": 'Bearer ' + sessionStorage.getItem('user_token')
                        },
                    }
                );
                if (response.data.status === true) {
                    $('.loader').fadeOut()
                    this.products_cart = response.data.data.products

                    for (let i = 0; i < this.products_cart.length; i++) {
                        this.products_cart[i].product_type = 1;
                    }
                    this.cards_cart = response.data.data.cards
                    for (let i = 0; i < this.cards_cart.length; i++) {
                        this.cards_cart[i].product_type = 2;
                    }
                    this.cart = this.products_cart.concat(this.cards_cart)
                }
            } catch (error) {
                console.error(error);
            }
        },
    },
    created() {
        this.fetchSubCategories(this.categoryId).then(() => {
            if (!this.subCategories || !this.subCategories.length)
                this.fetchProducts(this.categoryId)
                this.getCart()
        })
    },
    mounted() {
        $(`.${this.$route.meta.category_path}`).addClass('active')
        $(`.${this.$route.meta.category_path}`).siblings().removeClass('active')
    },
}
</script>