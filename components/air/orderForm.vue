<template>
  <div class="main">
    <div class="air-column">
      <h2>乘机人</h2>
      <el-form class="member-info">
        <div class="member-info-item" v-for="(item,index) in users" :key="index">
          <el-form-item label="乘机人类型">
            <el-input placeholder="姓名" class="input-with-select" v-model="item.username">
              <el-select slot="prepend" value="1" placeholder="请选择">
                <el-option label="成人" value="1"></el-option>
              </el-select>
            </el-input>
          </el-form-item>

          <el-form-item label="证件类型">
            <el-input placeholder="证件号码" class="input-with-select" v-model="item.id">
              <el-select slot="prepend" value="1" placeholder="请选择">
                <el-option label="身份证" value="1" :checked="true"></el-option>
              </el-select>
            </el-input>
          </el-form-item>

          <span class="delete-user" @click="handleDeleteUser()">-</span>
        </div>
      </el-form>

      <el-button class="add-member" type="primary" @click="handleAddUsers">添加乘机人</el-button>
    </div>

    <div class="air-column">
      <h2>保险</h2>
      <div>
        <div class="insurance-item" v-for="(item,index) in detail.insurances" :key="index">
          <el-checkbox :label="item.id" v-model="insurances" border>
          {{`${item.type}：￥${item.price}/份×1  最高赔付${item.compensation}万`}}
          </el-checkbox>
        </div>
      </div>
    </div>

    <div class="air-column">
      <h2>联系人</h2>
      <div class="contact">
        <el-form label-width="60px">
          <el-form-item label="姓名">
            <el-input v-model="contactName"></el-input>
          </el-form-item>

          <el-form-item label="手机">
            <el-input placeholder="请输入内容" v-model="contactPhone">
              <template slot="append">
                <el-button @click="handleSendCaptcha">发送验证码</el-button>
              </template>
            </el-input>
          </el-form-item>

          <el-form-item label="验证码">
            <el-input v-model="captcha"></el-input>
          </el-form-item>
        </el-form>
        <el-button type="warning" class="submit" @click="handleSubmit">提交订单</el-button>
      </div>
    </div>
    <!-- 调用总价格 让computed会执行 -->
    <span v-show="false">{{allPrice}}</span>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //机票详情
      detail: {
        insurances:[],
        seat_infos:{}
      },
      //乘机人是一个数据列表 初始化的时候需要添加一位乘机人
      users: [
        {
          username: "",
          id: ""
        }
      ],
      //保险id的集合
      insurances: [],
      contactName: "", //联系人
      contactPhone: "", //联系电话
      invoice: "", //是否要开发票 写死
      captcha: "" //验证码
    };
  },
  computed:{
      //计算总价格
      allPrice(){
         // 如果接口还没请求回来，直接返回
          if(!this.detail.seat_infos) return;
          //总价格初始值
          let price = 0;
          let len = this.users.length;
          //加上单价
          price += this.detail.seat_infos.org_settle_price * len;
          //保险费
          this.insurances.forEach(v=>{
              price += this.detail.insurances[v-1].price * len
          });
          //燃油费
          price += this.detail.airport_tax_audlet * len;

          //触发设置总金额事件
          this.$emit("getAllPrice",price)

          return price;
      }
  },
  methods: {
    // 添加乘机人
    handleAddUsers() {
      //给user添加乘机人
      this.users.push({
        username: "",
        id: ""
      });
    },

    // 移除乘机人
    handleDeleteUser(index) {
      //移出乘机人
      this.users.splice(index, 1);
    },

    // 发送手机验证码
    async handleSendCaptcha() {
      //先判断是否有手机号
      if (!this.contactPhone) {
        this.$message.error("手机号码不能为空");
        return;
      }
      const res = await this.$store.dispatch(
        "user/sendCaptcha",
        this.contactPhone
      );
      this.$message.success(`当前的验证码：` + res.data.code);
    },

    // 提交订单
    handleSubmit() {
      const data = {
        users: this.users,
        insurances: this.insurances,
        contactName: this.contactName, //联系人
        contactPhone: this.contactPhone, //联系电话
        invoice: this.invoice, //是否要开发票 写死
        captcha: this.captcha, //验证码
        seat_xid: this.$route.query.seat_xid,
        air: this.$route.query.id
      };
      const {
        user: { userInfo }
      } = this.$store.state;
      
      this.$axios({
        url: "/airorders",
        method: "POST",
        headers: {
          Authorization: `Bearer ${this.$store.state.user.userInfo.token}`
        },
        data
      }).then(res => {
          const {data} = res.data
          this.$message({
          message: "正在生成订单！请稍等",
          type: "success"
        });

          // 跳转到付款页
          this.$router.push({
            path: "/air/pay",
            query:{
              id:data.id
            }
          });
        })
        
        
    }
  },
  mounted() {
    const { id, seat_xid } = this.$route.query;
    //请求机票详情
    this.$axios({
      url: "/airs/" + id,
      params: {
        seat_xid
      }
    }).then(res => {
      this.detail = res.data;
      //把detail返回给父组件
      this.$emit("getDetail",this.detail)
    });
  }
};
</script>

<style scoped lang="less">
.air-column {
  border-bottom: 1px #ddd dashed;
  padding-bottom: 20px;
  margin-bottom: 20px;
}

.air-column h2 {
  margin-bottom: 20px;
  font-size: 22px;
  font-weight: normal;
}

/deep/ .el-select .el-input {
  width: 130px;
}

.input-with-select {
  width: 590px;
}

.input-with-select /deep/ .el-input-group__prepend {
  background-color: #fff;
}
.member-info /deep/ .el-form-item {
  margin-bottom: 0;
}

.member-info-item {
  border-bottom: 1px #eee dashed;
  padding-bottom: 20px;
  position: relative;

  &:first-child {
    .delete-user {
      display: none;
    }
  }
}

.add-member {
  margin-top: 20px;
}

.delete-user {
  display: block;
  background: #ddd;
  width: 16px;
  height: 16px;
  font-size: 14px;
  text-align: center;
  line-height: 16px;
  color: #fff;
  cursor: pointer;
  border-radius: 50px;
  position: absolute;
  right: -30px;
  top: 50%;
}

.insurance {
  > div {
    margin-top: 10px;
  }
}

.insurance-item {
  margin-bottom: 20px;
}

.contact {
  /deep/ .el-input {
    width: 50%;
  }
}

.submit {
  margin: 50px auto;
  display: block;
  width: 250px;
  height: 50px;
}
</style>