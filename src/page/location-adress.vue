<template>
    <div id="container">
        <div id="Purchase-history-container">
            <!-- <menuComponent></menuComponent> -->
            <div id="location-right">
                <h1 class="title">ที่อยู่</h1>
                <div id="form">
                    <form v-if="!isSaved" action="">
                        <div class="row">
                            <div class="row-item">
                                <label for="">ชื่อ-นามสกุล</label>
                                <input type="text" v-model="name">
                            </div>
                            <div class="row-item">
                                <label for="">เบอร์</label>
                                <input type="text" v-model="number">
                            </div>
                            <div class="row-item">
                                <label for="">แขวง/ตำบล</label>
                                <input type="text" v-model="Subdistrict">
                            </div>
                        </div>
                        <div class="row">
                            <div class="row-item">
                                <label for="">เขต/อำเภอ</label>
                                <input type="text" v-model="district">
                            </div>
                            <div class="row-item">
                                <label for="">จังหวัด</label>
                                <input type="text" v-model="province">
                            </div>
                            <div class="row-item">
                                <label for="">รหัสไปรษณีย์</label>
                                <input type="text" v-model="postalCode">
                            </div>
                        </div>
                        <div class="row">
                            <div class="row-item-detail">
                                <label for="">บ้านเลขที่, หมู่, ถนน, ซอย</label>
                                <input type="text" v-model="address" class="detail">
                            </div>
                            
                        </div>
                    </form>

                    <div v-else>
                        <!-- Display saved address data -->
                        <div class="row">
                            <div class="row-item2">
                                <strong>ชื่อ-นามสกุล: </strong>{{ name }}
                            </div>
                            <div class="row-item2">
                                <strong>เบอร์โทร: </strong>{{ number }}
                            </div>
                            <div class="row-item2">
                                <strong>เขต/อำเภอ: </strong>{{ Subdistrict }}
                            </div>
                        </div>
                        <div class="row">
                            <div class="row-item2">
                                <strong>เขต/อำเภอ: </strong>{{ district }}
                            </div>
                            <div class="row-item2">
                                <strong>จังหวัด: </strong>{{ province }}
                            </div>
                            <div class="row-item2">
                                <strong>รหัสไปรษณีย์: </strong>{{ postalCode }}
                            </div>
                            
                        </div>
                        <div class="row">
                            <div class="row-item2">
                                <strong>บ้านเลขที่, หมู่, ถนน, ซอย: </strong>{{ address }}
                            </div>
                            
                        </div>
                    </div>
                </div>
                <div id="map-container">
                    <mapAdress></mapAdress>
                </div>
                <!-- <div>
                    <router-link to="/AddressSeller">
                        <button>ยกเลิก</button>
                    </router-link>
                </div> -->
                <div id="location-button">
                    <router-link to="/AddressSeller">
                        <button>ยกเลิก</button>
                    </router-link>
                    <router-link to="/AddressSeller">
                        <button v-if="!isSaved && !hasAddress" @click="createAddress">บันทึก</button>
                    </router-link>
                    <router-link to="/AddressSeller">
                        <button v-if="!isSaved && hasAddress" @click="updateAddress" class="update">อัปเดต</button>
                    </router-link>
                    <button v-if="isSaved" @click="editAddress" class="edit">แก้ไข</button>
                </div>
            </div>
        </div>
    </div>
    <footerComponent></footerComponent>
</template>

<script>
import mapAdress from '../components/account/map-adress.vue';
import axios from 'axios';

export default {
    components: {
        mapAdress
    },
    data() {
        return {
            name: "",
            Subdistrict: "",
            district: "",
            postalCode: "",
            province: "",
            number: "",
            address: "",
            userEmail: '',
            // ispost: false,
            isSaved: false,
            hasAddress: false
        }
    },
    mounted() {
        this.getAddress();
    },
    methods: {
        createAddress() {
            const user = JSON.parse(localStorage.getItem('user'));
            const userEmail = user.email;
            const addressData = {
                name: this.name,
                Subdistrict: this.Subdistrict,
                district: this.district,
                postalCode: this.postalCode,
                province: this.province,
                address: this.address,
                email: userEmail,
                number: this.number
            };

            axios.post('http://localhost:8081/users/address', addressData)
                .then(response => {
                    console.log("Address created:", response.data);
                    this.isSaved = true;
                    this.hasAddress = true;
                })
                .catch(error => {
                    console.error("Error creating address:", error.response ? error.response.data : error);
                });
        },

        updateAddress() {
            const user = JSON.parse(localStorage.getItem('user'));
            const userEmail = user.email;
            const addressData = {
                name: this.name,
                Subdistrict: this.Subdistrict,
                district: this.district,
                postalCode: this.postalCode,
                province: this.province,
                address: this.address,
                email: userEmail,
                number: this.number
            };
            console.log("hasAddress", this.hasAddress);

            axios.put('http://localhost:8081/users/address', addressData)
                .then(response => {
                    console.log("Address updated:", response.data);
                    this.isSaved = true;
                })
                .catch(error => {
                    console.error("Error updating address:", error.response ? error.response.data : error);
                });
        },

        getAddress() {
            const user = JSON.parse(localStorage.getItem('user'));
            const userEmail = user.email;

            axios.get(`http://localhost:8081/users/address?email=${userEmail}`)
                .then(response => {
                    if (response.data) {
                        const addressData = response.data;
                        this.name = addressData.name;
                        this.Subdistrict = addressData.Subdistrict,
                        this.district = addressData.district;
                        this.postalCode = addressData.postalCode;
                        this.province = addressData.province;
                        this.address = addressData.address;
                        this.number = addressData.number;
                        this.isSaved = true;
                        if (addressData.name !== undefined && addressData.email === userEmail) {
                            this.hasAddress = true;
                        }
                    } else {
                        this.hasAddress = false;
                    }
                    console.log("hasAddress", this.hasAddress);
                })

                .catch(error => {
                    console.error("Error retrieving address:", error.response ? error.response.data : error);
                });
        },

        editAddress() {
            this.isSaved = false;
        }
    }

};
</script>

<style scoped>
* {
    font-family: "Noto Sans Thai", sans-serif;
    font-weight: 500;
    font-style: normal;
}

#Purchase-history-container {
    width: 800px;
    height: 690px;
    overflow: hidden;
    /* background-color: #1f1b3a; */
    background: white;
    display: flex;
    margin-top: 4rem;
}

#container {
    height: auto;
    display: flex;
    width: 100vw;
    flex-direction: column;
    align-items: center;
    /* margin-bottom: 30px; */
}

#location-button {
    margin-top: 1.5rem;
    width: 900px;
    display: flex;
    gap: 1rem;
    /* justify-content: flex-end; */
}

#location-button button {
    padding: 7px 10px 7px 10px;
    font-size: 17px;
    cursor: pointer;
    border-radius: 5px;
    border: 1px solid gray;
    background: white;
    transition: 0.3s;
}

#location-button button:hover {
    background-color: #ffa200;
    border: none;
    color: white;
}

#item-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

form {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: -20px;
}

#form input {
    border: 1px solid rgb(203, 203, 203);
    border-radius: 2px;
    padding: 5px
}

.row {
    display: flex;
    gap: 40px;
    /* justify-content: flex-end; */
}

.row-item {
    display: flex;
    flex-direction: column;
}

.row-item2 {
    margin-top: 40px;
}

.row-item label {
    font-size: 18px;
}

.detail {
    display: flex;
    flex-direction: column;
    width: 500px;
    height: 28px;
}

.row-item input {
    width: 200px;
    height: 28px;
}

.icon {
    font-size: 30px;
}

#location-right {
    padding: 20px;
    width: 800px;
    height: 700px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    background-color: white;
}

#map-container {
    width: 700px;
    display: flex;
    justify-content: flex-start;
    margin-top: -10px;
}

#container {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    overflow: hidden;
    margin-top: 2rem;
    /* ระยะห่างด้านบน */
}

#Purchase-history-container {
    width: 800px;
    /* กำหนดขนาดเท่ากัน */
    height: 630px;
    /* background-color: #1f1b3a; */
    display: flex;
    align-items: stretch;
    margin-top: 2rem;
    /* ระยะห่างด้านบนเท่ากัน */
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.title {
    font-weight: 700;
}

.edit {
    border: 1px solid red;
    color: red;
}

.update {
    border: 1px solid green;
    color: green;
}

</style>