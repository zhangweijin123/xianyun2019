<template>
    <div class="container">
        <el-row type="flex" justify="space-between">
            <!-- 订单表单 -->
            <div class="main">
                <OrderForm @getDetail="getDetail" @getAllPrice="getAllPrice"/>
            </div>

            <!-- 侧边栏 -->
            <div class="aside">
                <OrderAside :data="detail" :allPrice="allPrice"/>    
            </div>
        </el-row>
    </div>
</template>

<script>
import OrderForm from "@/components/air/orderForm.vue"
import OrderAside from "@/components/air/orderAside.vue"
export default {
    data(){
        return{
            detail:{
                seat_infos:{}
            },

            allPrice:0  //总价格初始值为0
        }
    },
    components:{
        OrderForm,
        OrderAside
    },

    methods:{
        //获取机票详情的方法
        getDetail(detail){
            this.detail = detail;
        },

        //获取总价格
        getAllPrice(price){
            this.allPrice = price;
        }
    }
}
</script>

<style lang="less" scoped>
    .container{
        width:1000px;
        margin:20px auto;
    }
    
    /*aside*/
    .aside{
        width: 350px;
        height: fit-content;
        border:1px #ddd solid;
    }
</style>