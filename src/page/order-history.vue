<template>
    <div id="container">
        <div id="Purchase-history-container">
            <menuComponent></menuComponent>
            <div id="Purchase-history-right">
                <h1>ประวัติสินค้าที่ถูกซื้อ</h1>
                <div class="fillter">
                    <div class="search-container">
                        <label for="searchProduct">ค้นหาสินค้า:</label>
                        <select v-model="searchProduct" @change="filterProducts">
                            <option value="">-- ทั้งหมด --</option>
                            <option v-for="productName in uniqueProductNames" :key="productName" :value="productName">
                                {{ productName }}
                            </option>
                        </select>
                        <label for="startDate">วันที่เริ่มต้น:</label>
                        <input type="date" v-model="startDate" @change="filterProducts">
                        <label for="endDate">วันที่สิ้นสุด:</label>
                        <input type="date" v-model="endDate" @change="filterProducts">
                        <button @click="toggleSortOrder" class="sort-button">
                            {{ sortOrder === 'desc' ? 'เรียงจากใหม่ไปเก่า' : 'เรียงจากเก่าไปใหม่' }}
                        </button>
                    </div>
                </div>

                <div id="item-container">
                    <div v-for="product in filteredAndSortedHistory" :key="product.productId" class="item">
                        <div class="line"></div>
                        <div class="items">
                            <input type="checkbox" v-model="product.checkbox" class="checked"
                                @change="setSelectItem(product)">
                            <img :src="product.image" alt="" style="width: 50px; height: 50px; object-fit: cover;">
                            <div class="item-1">
                                <p><span>{{ product.nameProduct }}</span></p>
                            </div>
                            <div class="item-2">
                                <p>ตัวเลือก:<span>{{ product.productTypes }}</span></p>
                                <p>จำนวน:<span>{{ product.quantity }}</span></p>
                                <p>{{ calculateLinePrice(product).toFixed(2) }} บาท</p>
                                <p><span>{{ new Date(product.createdAt).toLocaleString('th-TH') }}</span></p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="item-button">
                    <!-- <button @click="handleClick">สั่งซื้ออีกครั้ง</button> -->
                    <button @click="cancelOrder">ลบรายการประวัติสินค้า</button>
                </div>
            </div>
        </div>
    </div>
    <footerComponent></footerComponent>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            history: [],
            currentPage: 1,
            itemsPerPage: 12,
            totalItems: 0,
            shops: [],
            products: [],
            selectedItems: [],
            shopId: "",
            sortOrder: 'desc',
            searchProduct: '',
            startDate: '',  
            endDate: '',   
            // selectedItems: [],
        };
    },
    computed: {
        filteredAndSortedHistory() {
            let filteredHistory = this.history;

            // กรองสินค้าตามชื่อที่เลือก
            if (this.searchProduct) {
                filteredHistory = filteredHistory.filter(
                    product => product.nameProduct === this.searchProduct
                );
            }

            // กรองสินค้าตามวันที่เริ่มต้นและวันที่สิ้นสุด
            if (this.startDate) {
                const start = new Date(this.startDate);
                filteredHistory = filteredHistory.filter(
                    product => new Date(product.createdAt) >= start
                );
            }
            if (this.endDate) {
                const end = new Date(this.endDate);
                filteredHistory = filteredHistory.filter(
                    product => new Date(product.createdAt) <= end
                );
            }

            // เรียงลำดับตามวันที่
            return filteredHistory.slice().sort((a, b) => {
                const dateA = new Date(a.createdAt);
                const dateB = new Date(b.createdAt);
                return this.sortOrder === 'desc' ? dateB - dateA : dateA - dateB;
            });
        },
        uniqueProductNames() {
            const names = this.products.map(product => product.nameProduct);
            return [...new Set(names)]; // กำจัดชื่อที่ซ้ำออก
        },
        filteredHistory() {
            if (this.searchProduct) {
                // กรองสินค้าตามชื่อที่เลือกจาก dropdown
                return this.history.filter(product => product.nameProduct === this.searchProduct);
            }
            return this.history;
        },
        totalPages() {
            return Math.ceil(this.totalItems / this.itemsPerPage);
        },
        paginatedProducts() {
            const start = (this.currentPage - 1) * this.itemsPerPage;
            const end = start + this.itemsPerPage;
            return this.history.slice(start, end);
        },
        sortedHistory() {
            return this.history.slice().sort((a, b) => {
                const dateA = new Date(a.createdAt);
                const dateB = new Date(b.createdAt);
                return this.sortOrder === 'desc' ? dateB - dateA : dateA - dateB;
            });
        },
    },
    methods: {
        toggleSortOrder() {
            this.sortOrder = this.sortOrder === 'desc' ? 'asc' : 'desc';
        },
        totalPrice() {
            if (this.selectedItems.length > 0) {
                return this.calculateLinePrice(this.selectedItems[0]).toFixed(2);
            }
            return 0;
        },
        calculateLinePrice(product) {
            return product.price * product.quantity;
        },
        filteredProducts(shopId) {
            return this.products.filter(product => product.shopId === shopId);
        },
        async loadHistory() {
            try {
                const response = await axios.get(`http://localhost:8081/products/gethistorySeller`);
                console.log("this.shopId", this.shopId);

                this.history = response.data.filter(entry => entry.shopId === this.shopId);
                // this.totalItems = this.history.length;
                console.log('Loaded history:', this.history);

                // console.log('Total items in history:', this.totalItems);
            } catch (error) {
                console.error('Error fetching history:', error);
            }
        }, async loadshop() {
            try {
                const user = JSON.parse(localStorage.getItem('user'));
                if (!user) throw new Error('User not found in localStorage');
                const userEmail = user.email;
                if (!userEmail) throw new Error('User email not found');

                const response = await axios.get(`http://localhost:8081/shop/shops`);
                console.log("sss", response.data);

                const shopp = Array.isArray(response.data.data)
                    ? response.data.data.filter(entry => entry.email === userEmail)
                    : [];

                if (shopp.length > 0) {
                    this.shopId = shopp[0].shopId;
                }

                console.log("this.shopId:", this.shopId);

            } catch (error) {
                console.error('Error fetching shops:', error);
            }
        }
        ,
        reorder(product) {
            console.log('Reordering product:', product);
        },
        async cancelOrder() {
            const itemsToRemove = this.selectedItems.filter(product => product.checkbox);
            try {

                for (const product of itemsToRemove) {
                    console.log('History ID to delete:', product.Historyid);
                    await axios.delete(`http://localhost:8081/products/delHistorySellrt/${product.Historyid}`);
                    console.log(`Deleted product with ID: ${product.productId}`);
                }
                await this.loadHistory();
                window.location.reload();
            } catch (error) {
                console.error('Error canceling orders:', error);
            }
        },
        gotoPage(page) {
            this.currentPage = page;
        },
        async getProducts() {
            try {
                const response = await axios.get("http://localhost:8081/products/getHistory");
                this.products = response.data;
                console.log("Fetched Products:", this.products);
            } catch (error) {
                console.error("Error fetching products:", error.response ? error.response.data : error);
            }
        },
        setSelectItem(product) {
            if (product.checkbox) {
                this.selectedItems.push(product);

            } else {
                const index = this.selectedItems.findIndex(item => item.id === product.id);
                if (index !== -1) {
                    this.selectedItems.splice(index, 1);
                }
            }
            console.log("Selected items:", this.selectedItems);
        },

    },
    async mounted() {
        this.loadshop();
        await this.getProducts();
        await this.loadHistory();

    }
};
</script>




<style scoped>
* {
    font-family: "Noto Sans Thai", sans-serif;
    font-weight: 500;
    font-style: normal;
}

#container {
    height: auto;
    display: flex;
    width: 100vw;
    flex-direction: column;
    align-items: center;
    background-color: #ffffff;
}

#Purchase-history-container {
    width: 1100px;
    height: 100%;
    /* กำหนดขนาดเท่ากัน */
    background-color: #1f1b3a;
    display: flex;
    align-items: stretch;
    margin-top: 2rem;
    /* ระยะห่างด้านบนเท่ากัน */
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    margin-left: -15px;
    margin-top: 63px;
}

#Purchase-history-right {
    /* padding: 20px;
    flex: 1;
    background-color: #F4F4F5;
    display: flex;
    flex-direction: column;
    align-items: center; */
    width: 1100px;
    height: auto;
    min-height: 590px;
    padding: 20px;
    background-color: #ffffff;
    display: flex;
    /* margin-top: -15px;
    margin-right: -38px; */
    flex-direction: column;
    gap: 1.5rem;
}

h1 {
    margin-top: 20px;
    font-weight: 700;
}

#item-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 100%;
    margin-top: 20px;
}

.item {
    width: 100%;
    background-color: #ffffff;
    padding: 15px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.line {
    width: 100%;
    height: 1px;
    background-color: rgb(179, 172, 172);
    margin: 10px 0;
}

.items {
    display: flex;
    gap: 20px;
    align-items: center;
    width: 100%;
}

.checked {
    width: 25px;
    height: 25px;
}

.item-1,
.item-2 {
    flex: 1;
}

.item-1 span,
.item-2 span {
    font-size: 16px;
}

.item-2 {
    display: flex;
    flex-direction: column;
    gap: 5px;
}

.item-button {
    display: flex;
    width: 100%;
    margin-top: 20px;
    justify-content: flex-end;
    gap: 10px;
}

.item-button button {
    padding: 10px 20px;
    background-color: #ffffff;
    border: 2px solid #ffbd4c;
    color: #df8d00;
    border-radius: 4px;
    cursor: pointer;
    transition: 0.3s;
}

.item-button button:hover {
    background-color: #ffa8a8;
    border: 2px solid #fe6565;
    color: black;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.4);
}

#Purchase-history-pagination {
    display: flex;
    gap: 10px;
    width: 100%;
    justify-content: flex-end;
    margin-top: 20px;
}

#Purchase-history-pagination button {
    width: 30px;
    height: 30px;
    background-color: #ffffff;
    border: 1px solid #ddd;
    border-radius: 50%;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

#Purchase-history-pagination button:hover {
    background-color: #fde0ae;
}

/* .fillter {
    width: 95%;
    height: 35px;
    background-color: #ffffff;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 5px 10px;
    margin-top: -30px;
} */

.search-container {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
    background-color: #f8f8f8;
    padding: 10px;
    border-radius: 8px;
    margin-top: -30px;
    font-size: 14px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

.search-container label {
    font-weight: bold;
    color: #333;
}

.search-container input[type="date"] {
    width: 110px;
}

.search-container select,
.search-container input[type="date"],
.search-container .sort-button {
    padding: 5px;
    font-size: 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: #fff;
    color: #333;
    cursor: pointer;
}

.search-container .sort-button {
    background-color: #ee9700;
    color: white;
    border: none;
    cursor: pointer;
    padding: 5px 10px;
    transition: background-color 0.3s;
}

.search-container .sort-button:hover {
    background-color: #ffc86a;
    color: black;
}

</style>