<template>
    <div class="search-form">

        <!-- 头部tab切换 -->
        <el-row type="flex" class="search-tab">
            <span v-for="(item, index) in tabs" :key="index"
            @click="handleSearchTab(item, index)"
            :class="{active: index === currentTab}">
                <i :class="item.icon"></i>{{item.name}}
            </span>
        </el-row>

        <el-form class="search-form-content" ref="form" label-width="80px">
            <el-form-item label="出发城市">
                <!-- fetch-suggestions 返回输入建议的方法 -->
                <!-- select 点击选中建议项时触发 -->
                <el-autocomplete
                :fetch-suggestions="queryDepartSearch"
                placeholder="请搜索出发城市"
                @select="handleDepartSelect"
                class="el-autocomplete"
                v-model="form.departCity"
                @blur="handleDepartBlur"
                ></el-autocomplete>
            </el-form-item>
            <el-form-item label="到达城市">
                <el-autocomplete
                :fetch-suggestions="queryDestSearch"
                placeholder="请搜索到达城市"
                @select="handleDestSelect"
                class="el-autocomplete"
                v-model="form.destCity"
                @blur="handleDestBlur"
                ></el-autocomplete>
            </el-form-item>
            <el-form-item label="出发时间">
                <!-- change 用户确认选择日期时触发 -->
                <el-date-picker type="date" 
                placeholder="请选择日期" 
                style="width: 100%;"
                @change="handleDate"
                v-model="form.departDate">
                </el-date-picker>
            </el-form-item>
            <el-form-item label="">
                <el-button style="width:100%;" 
                type="primary" 
                icon="el-icon-search"
                @click="handleSubmit">
                    搜索
                </el-button>
            </el-form-item>
            <div class="reverse">
                <span @click="handleReverse">换</span>
            </div>
        </el-form>  
      </div>
</template>

<script>
import moment from "moment";
export default {
    data(){
        return {
            tabs: [
                {icon: "iconfont icondancheng", name: "单程"},
                {icon: "iconfont iconshuangxiang", name: "往返"}
            ],
            currentTab: 0,
            //参数
            form:{
                departCity:"",
                departCode:"",
                destCity:"",
                destCode:"",
                departDate:"",
            },
            //存放newData的城市数组
            cities:[]
        }
    },
    methods: {
        // tab切换时触发
        handleSearchTab(item, index){
            if(index===1){
                this.$alert('目前不支持往返','提示',{
                    confirButtonText:'确定',
                    type:"warning"
                })
            }
        },
        
        // 出发城市输入框获得焦点时触发
        // value 是选中的值，cb是回调函数，接收要展示的列表
        queryDepartSearch(value, cb){
            //请求为空的时候不请求
            if(!value){
                cb([])
                return 
            }
            //获取后台的城市数据
            this.$axios({
                url:"/airs/city?name=" + value,
            }).then(res=>{
                //data是后台返回来的城市数组 没有value属性
                const {data} = res.data;
                //循环遍历数组添加value这一属性
                const newData = data.map(v=>{
                    v.value = v.name.replace("市" , "")
                    return v;
                })
                //把newData的值赋值给cities
                this.cities = newData;
                cb(newData);
            })
        },

        // 目标城市输入框获得焦点时触发
        // value 是选中的值，cb是回调函数，接收要展示的列表
        queryDestSearch(value, cb){
            this.queryDepartSearch(value, cb)
        },

        //失去焦点的时候默认选中第一个
        handleDepartBlur(){
            if(this.cities.length > 0){
            this.form.departCity = this.cities[0].value
            this.form.departCode = this.cities[0].sort
            }
        },
        handleDestBlur(){
            if(this.cities.length > 0){
                this.form.destCity = this.cities[0].value
            this.form.destCode = this.cities[0].sort
            }
        },
       
        // 出发城市下拉选择时触发把数据储存
        handleDepartSelect(item) {
            this.form.departCity = item.value
            this.form.departCode = item.sort
        },

        // 目标城市下拉选择时触发把数据储存
        handleDestSelect(item) {
            this.form.destCity = item.value
            this.form.destCode = item.sort
        },

        // 确认选择日期时触发
        handleDate(value){
           this.form.departDate = moment(value).format("YYYY-MM-DD")
        },

        // 触发和目标城市切换时触发
        handleReverse(){
            const {departCity,departCode,destCity,destCode} = this.form

            this.form.departCity = destCity;
            this.form.departCode = destCode;
            this.form.destCity = departCity;
            this.form.destCode = departCode
        },

        // 提交表单时触发
        handleSubmit(){
            //自定义验证
            const rules = {
                departCity:{
                    //message是错误信息，value是对应表单中的值
                    message:"请输入出发城市",value:this.form.departCity
                },
                destCity:{
                    message:"请输入到达城市",value:this.form.destCity
                },
                departDate:{
                    message:"请选择出发时间",value:this.form.departDate
                }
            }
            //循环rules这个对象 判断对象属性的value如果是空 打印出message错误信息
            let valid = true;


            Object.keys(rules).forEach(v=>{
                //只要有一次验证不通过 后台验证不用在执行
                if(!valid) return;

                const {message,value} = rules[v];

                //对象属性的value如果是空的
                if(!value){
                    this.$message.error(message)
                    //验证不通过
                    valid = false;
                }
            })

            if(!valid) return

            this.$router.push({
                path:"air/flights",
                query:this.form
            });

            //把搜索到的值保存到store
            this.$store.commit("air/setHistory",this.form)
        }
    },
    
}
</script>

<style scoped lang="less">
.search-form{
    border:1px #ddd solid;
    border-top:none;
    width:360px;
    height:350px;
    box-sizing: border-box;
}

.search-tab{
  span{
    display: block;
    flex:1;
    text-align: center;
    height:48px;
    line-height: 42px;
    box-sizing: border-box;
    border-top:3px #eee solid;
    background:#eee;
  }

  .active{
    border-top-color: orange;
    background:#fff;
  }

  i{
    margin-right:5px;
    font-size: 18px;

    &:first-child{
      font-size:16px;
    }
  }
}

.search-form-content{
  padding:15px 50px 15px 15px;
  position: relative;

  .el-autocomplete{
    width: 100%;
  }
}

.reverse{
  position:absolute;
  top: 35px;
  right:15px;

  &:after,&:before{
      display: block;
      content: "";
      position: absolute;
      left:-35px;
      width:25px;
      height:1px;
      background:#ccc;
  }

  &:after{
      top:0;
    }

    &:before{
      top:60px;
    }

  span{
    display: block;
    position:absolute;
    top: 20px;
    right:0;
    font-size:12px;
    background: #999;
    color:#fff;
    width:20px;
    height:20px;
    line-height: 18px;
    text-align: center;
    border-radius: 2px;
    cursor: pointer;

    &:after,&:before{
      display: block;
      content: "";
      position: absolute;
      left:10px;
      width:1px;
      height:20px;
      background:#ccc;
    }

    &:after{
      top:-20px;
    }

    &:before{
      top:20px;
    }
  }
}
</style>