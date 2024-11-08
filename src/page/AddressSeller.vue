<template>
    <div class="checkout">
        <h1>การชำระเงิน</h1>

        <!-- ส่วนที่อยู่ -->
        <div class="address-section">
            <h2>ที่อยู่การจัดส่ง</h2>
            <div v-if="Adressfilter.length" class="address">
                <div v-for="(address, index) in Adressfilter" :key="index" class="address-item">
                    <p>{{ address.name }}</p>
                    <p>{{ address.number }},</p>
                    <p>{{ address.address }},</p>
                    <p>ตำบล{{ address.Subdistrict }},</p>
                    <p>อำเภอ{{ address.district }}, </p>
                    <p>จังหวัด{{ address.province }}</p>
                    <p>{{ address.postalCode }}</p>
                    <!-- <p>อีเมล: {{ address.email }}</p> -->
                </div>
                <router-link to="/users/locationAdress">
                    <p>เปลี่ยน</p>
                </router-link>
            </div>
            <router-link v-else to="/users/locationAdress">
                <p>กรุณากรอกข้อที่อยู่</p>
            </router-link>
        </div>

        <div class="product-details-section">
            <div class="details">
                <h2>สั่งซื้อสินค้าแล้ว</h2>
            </div>

            <div v-for="product in this.Orderfilter" :key="product.productId" class="itemm">
                <div class="line"></div>
                <div class="items">
                    <img :src="product.image" alt="" style="width: 70px; height: 70px; object-fit: cover;">
                    <div class="item-1">
                        <p><span>{{ product.nameProduct }}</span></p>
                        <div class="typeProduct">
                            <p>ตัวเลือก : </p>
                            <span>{{ product.productTypes }}</span>
                        </div>
                    </div>
                    <div class="item-2">
                        <div class="productprice">
                            <p>ราคาต่อหน่วย</p>
                            <span>{{ product.price }}</span>
                        </div>
                        <div class="quantity">
                            <p>จำนวน</p>
                            <span>{{ product.quantity }}</span>
                        </div>
                        <div class="price">
                            <p>ราคา</p>
                            <span>{{ calculateLinePrice(product).toFixed(2) }} บาท</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- <div class="product-details-section">
            <receiptComponent :orderFilter="Orderfilter" />
        </div> -->
        <div v-for="product in this.Orderfilter" :key="product.productId" class="payment-section">
            <div class="payment">
                <h2>การชำระเงิน</h2>
                <button @click="toggleButtonColor">VISA</button>
            </div>
            <div class="payme-amount">
                <div class="payme-amount-1">
                    <span>รวมการสั่งซื้อ</span>
                    <span>VAT 7%</span>
                    <span>ยอดชำระเงินทั้งหมด</span>
                </div>
                <div class="payme-amount-2">
                    <span>฿{{ calculateLinePrice(product).toFixed(2) }}</span>
                    <span>฿{{ (calculateLinePrice(product).toFixed(2)) * 7 / 100 }}</span>
                    <span>฿{{ (parseFloat(calculateLinePrice(product).toFixed(2)) + parseFloat((calculateLinePrice(product).toFixed(2)) * 7 /100)).toFixed(2) }}</span>
                </div>
            </div>
            <button @click="handleClick()" :disabled="!Adressfilter.length" class="payment-button">ชำระเงิน</button>
        </div>
    </div>
</template>

<script>
// import receiptComponent from './page/receipt-component.vue';
// import receiptComponent from './receipt-component.vue';
import axios from 'axios';
import { jwtDecode } from 'jwt-decode';
// import { v4 as uuidv4 } from 'uuid';

export default {
    components: {
        // receiptComponent
    },
    data() {
        return {
            Adressfilter: [],
            Orderfilter: [],
            // Orderfilter: []
        };
    }, async mounted() {
        // this.handleRedirectToLocalhost();
        await this.loadCart()
        await this.loadOrder()
    },
    methods: {
        handleClick() {
            this.addToHistoryClicked();
            // this.$emit('orderCompleted', this.Orderfilter);
            this.payment();
        },
        toggleButtonColor(event) {
            if (event.target.style.backgroundColor === 'red' || event.target.style.color === 'red') {
                event.target.style.border = '';
                event.target.style.color = '';
            } else {
                event.target.style.border = '1px solid red';
                event.target.style.color = 'red';
            }
        },
        truncatedName(name) {
            if (name.length > 35) {
                // const firstLine = name.substring(0, 20);
                const secondLine = name.substring(0, 35) + "...";
                return `${secondLine}`;
            }
            return name;
        },
        addToHistoryClicked() {
            this.Orderfilter.forEach(product => {
                const calculatedPrice = this.calculateLinePrice(product).toFixed(2);
                const productData = {
                    productId: product.productId,
                    nameProduct: product.nameProduct,
                    price: calculatedPrice,
                    quantity: product.quantity,
                    image: product.image,
                    email: product.email,
                    shopId: product.shopId,
                    productTypes: product.productTypes,
                    Orderid: product.Orderid,
                };
                console.log("productData", productData.Orderid);
                // axios.put(`http://localhost:8081/selling/updateProduct/${productData.productId}`, {
                //     totalSell: parseInt(productData.quantity),
                //     totalPrice: parseFloat(productData.price) * parseInt(productData.quantity)
                // })
                //     .then(response => {
                //         console.log("Product added to history:", response.data);
                //     })
                //     .catch(error => {
                //         console.error("Error details:", error.response.data);
                //     });
                axios.post('http://localhost:8081/products/OrderHistory', {
                    Orderid: productData.Orderid,
                    productId: productData.productId,
                    nameProduct: productData.nameProduct,
                    image: productData.image,
                    email: productData.email,
                    shopId: productData.shopId,
                    price: productData.price,
                    quantity: productData.quantity,
                    productTypes: productData.productTypes,
                }).then(response => {
                    console.log("❤❤❤❤❤:", response.data);
                })
                    .catch(error => {
                        console.error("เกิดข้อผิดพลาดในการสร้าง Entry ใน history:", error, productData.Orderid);
                    });
                const user = JSON.parse(localStorage.getItem('user'));
                if (!user) {
                    throw new Error('User not found in localStorage');
                }
                const userEmail = user.email;
                if (!userEmail) {
                    throw new Error('User email not found');
                }
                const selectedIds = this.Orderfilter.map(order => order.Orderid).join(',');
                axios.post('http://localhost:8081/users/recipes', { email:userEmail,Orderid:selectedIds })
                    .then(response => {
                        console.log("⭕⭕⭕", response.data);
                    })
                    .catch(error => {
                        console.error("เกิดข้อผิดพลาดในการสร้าง recipes ใน recipes:", error.response ? error.response.data : error);
                    });
                axios.post('http://localhost:8081/products/createHistoryEntry', productData)
                    .then(response => {
                        console.log("สร้าง Entry ใน history สำเร็จ:", response.data);

                    })
                    .catch(error => {
                        console.error("เกิดข้อผิดพลาดในการสร้าง Entry ใน history:", error.response ? error.response.data : error);
                    });
                axios.post('http://localhost:8081/products/createHistoryEntrySeller', productData)
                    .then(response => {
                        console.log("สร้าง Entry ใน historySeller สำเร็จ:", response.data);
                    }).catch(error => {
                        console.error("เกิดข้อผิดพลาดในการสร้าง Entry ใน historySeller:", error.response ? error.response.data : error);
                    });
            });
        }
        ,
        payment() {
            // const selectedIds = this.Orderfilter.map(order => order.Orderid).join(',');
            // console.log("selectedIds", selectedIds);
            const totalAmount = this.Orderfilter
                .reduce((sum, order) => sum + this.calculateLinePrice(order), 0)
                .toFixed(2);
            // console.log("Total amount:", totalAmount);
            // const invoiceNo = uuidv4();

            axios.post('http://localhost:8081/2c2p/paymentToken', {
                // invoiceNo:selectedIds ,
                amount: totalAmount,
                // userDefined1: selectedIds
            })
                .then(paymentResponse => {
                    // console.log('Full payment response:', paymentResponse.data);
                    const payloadObject = paymentResponse.data;
                    const payload = payloadObject.payload?.payload?.toString();
                    // console.log('Payload received from 2C2P:', payload);

                    if (typeof payload === 'string') {
                        const decoded = jwtDecode(payload);
                        // console.log('Decoded JWT:', decoded);
                        const webPaymentUrl = decoded.webPaymentUrl;
                        if (webPaymentUrl) {
                            // this.clearCart();
                            window.location.href = webPaymentUrl;
                        } else {
                            alert('ไม่พบลิงก์สำหรับการจ่ายเงินใน Payload');
                        }
                    } else {
                        alert('เกิดข้อผิดพลาดในการประมวลผล payment token');
                    }
                })
                .catch(error => {
                    if (error.response) {
                        console.error('Error processing payment token:', error.response.data);
                    } else {
                        console.error('Error processing payment token:', error.message);
                    }
                    alert('เกิดข้อผิดพลาดในการประมวลผล payment token');
                });
        },
        calculateLinePrice(product) {
            return product.price * product.quantity;
        }, showReceipt(orderData) {
            this.orderData = orderData;
        },
        async loadOrder() {
            try {
                const user = JSON.parse(localStorage.getItem('user'));
                if (!user) {
                    throw new Error('User not found in localStorage');
                }
                const userEmail = user.email;
                if (!userEmail) {
                    throw new Error('User email not found');
                }

                const response = await axios.get('http://localhost:8081/products/getProductTestOrder');
                // console.log("response", response);

                const Adress = Array.isArray(response.data) ? response.data : [response.data];
                this.Orderfilter = Adress.filter(entry => entry.email === userEmail);
                console.log("this.Orderfilter", this.Orderfilter);
            } catch (error) {
                console.error('Error fetching cart items:', error);
            }
        },
        // async clearCart() {
        //     try {
        //         const user = JSON.parse(localStorage.getItem('user'));
        //         if (!user) {
        //             throw new Error('User not found in localStorage');
        //         }
        //         const userEmail = user.email;
        //         if (!userEmail) {
        //             throw new Error('User email not found');
        //         }

        //         await axios.delete('http://localhost:8081/products/DelProductTestOrder', {
        //             data: { email: userEmail }
        //         });

        //         // console.log('Cart items cleared for email:', userEmail);
        //     } catch (error) {
        //         console.error('Error clearing cart items:', error);
        //     }
        // },
        async loadCart() {
            try {
                const user = JSON.parse(localStorage.getItem('user'));
                if (!user) {
                    throw new Error('User not found in localStorage');
                }
                const userEmail = user.email;
                if (!userEmail) {
                    throw new Error('User email not found');
                }
                const response = await axios.get(`http://localhost:8081/users/address?email=${userEmail}`);
                const Adress = Array.isArray(response.data) ? response.data : [response.data];
                this.Adressfilter = Adress
                // console.log(this.Adressfilter);
            } catch (error) {
                console.error('Error fetching cart items:', error);
            }
        }
        ,
        // handleRedirectToLocalhost() {
        //     const currentUrl = window.location.href;
        //     const expectedLocalhostUrl = 'http://localhost:8080/AddressSeller';
        //     console.log('Current URL:', currentUrl);
        //     if (currentUrl !== expectedLocalhostUrl) {
        //         window.location.href = expectedLocalhostUrl;
        //     }
        // },
    }
};
</script>

<style scoped>
* {
    font-family: "Noto Sans Thai", sans-serif;
    font-weight: 500;
    font-style: normal;
}

.title {
    font-weight: 800;
}

.address {
    display: flex;
    font-size: 15px;
    font-weight: 400;
    margin-left: -20px;
    margin-top: -20px;
}

.address p:nth-child(1),
.address p:nth-child(2) {
    font-weight: 700;
}

.address-item {
    padding-left: 20px;
    display: flex;
    gap: 10px;
    width: 80%;
    margin-right: 20px;
}

.checkout {
    padding: 20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.line {
    width: 100%;
    height: 2px;
    background-color: rgb(179, 172, 172);
    margin: 20px 0 25px 0;
}

.product {
    border: 1px solid #ccc;
    padding: 10px;
    margin-bottom: 10px;
}

.address-section,
.product-details-section,
.payment-section {
    /* display: flex; */
    margin-bottom: 20px;
    background-color: #ffffff;
    padding: 1rem 1.5rem;
    width: 1100px;
    border-radius: 5px;
}

.address-section h2 {
    color: red;
    font-weight: 700;
}

.details {
    display: flex;
    align-items: center;
}

.item-1 {
    margin-left: 20px;
    display: flex;
    align-items: center;
    gap: 20px;
}

.typeProduct {
    display: flex;
    flex-direction: row;
    align-items: center;
    color: rgb(92, 92, 92);
    gap: 5px;
}

.items {
    display: flex;
    /* height: 88px; */
    align-items: center;
}

.item-2 {
    display: flex;
    gap: 40px;
    margin-top: -60px;
    align-items: center;
    margin-left: auto;
}

.payment-button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
    opacity: 0.6;
}

.payment-button {
    width: 200px;
    padding: 10px 20px;
    margin-top: 30px;
    margin-left: 77%;
    border: 1px solid rgb(255, 136, 0);
    background: white;
    color: rgb(255, 136, 0);
    border-radius: 5px;
    cursor: pointer;
    font-weight: 800px;
    font-size: 20px;
    transition: 0.3s;
}

.payment-button:hover {
    border: 1px solid rgb(255, 136, 0);
    background-color: rgb(255, 136, 0);
    color: #ffffff;
}

.payment-section {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.payment {
    display: flex;
    flex-direction: row;
    gap: 60px;
}

.payment button {
    width: 80px;
    height: 40px;
    margin-top: 10px;
    color: rgb(0, 0, 0);
    border: 1px solid rgb(169, 169, 169);
    background: #fff;
    cursor: pointer;
    transition: 0.3s;
}

.payment button:hover {
    border: 1px solid #d79d09;
    background-color: #ffe8ad;
    color: #000;
}

.payme-amount {
    display: flex;
    justify-content: flex-end;
    flex-direction: row;
    gap: 20px;
    /* align-items: flex-end; */
}

.payme-amount-1, 
.payme-amount-2 {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.payme-amount-1 {
    font-weight: 700;
    width: 150px;
}

.payme-amount-2 {
    min-width: 90px;
}

.payme-amount-2 span:nth-child(3) {
    font-weight: 800px;
    color: red;
    margin-top: -5px;
    font-size: 20px;
}

.change {
    text-decoration: none;
    transition: 0.3s;
}

.change:hover {
    color: red;
    text-decoration: underline;
}
</style>