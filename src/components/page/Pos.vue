<template>
  <div class="pos">
    <el-row>
      <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table border style="width:100%;" :data="tableData">
              <el-table-column label="商品名称" prop="goodsName"></el-table-column>
              <el-table-column label="量" prop="count" width="50"></el-table-column>
              <el-table-column label="金额" prop="price" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template scope="scope">
                  <el-button size="small" type="text" @click="deleteGoods(scope.row)">删除</el-button>
                  <el-button size="small" type="text" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
        <div class="total">
          <small>数量：</small>{{totalCount}} &nbsp;&nbsp;&nbsp;&nbsp;
          <small>金额:</small>{{totalMoney}}元
        </div>
        <div class="divBtn">
          <el-button type="warning">挂单</el-button>
          <el-button type="danger" @click="delAll()">删除</el-button>
          <el-button type="success" @click="checkOut()">结账</el-button>
        </div>

      </el-col>
      <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul class="clearfix">
              <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="goods-type clearfix">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" /></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <br/>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" /></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <br/>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" /></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <br/>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%" /></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <br/>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'pos',
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    }
  },
  created: function() {
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
      .then(response => {
        //console.log(response);
        this.oftenGoods = response.data;
      })
      .catch(error => {
        alert('网络错误，请稍后再试');
      })

    axios.get('http://jspang.com/DemoApi/typeGoods.php')
      .then(response => {
        this.type0Goods = response.data[0];
        this.type1Goods = response.data[1];
        this.type2Goods = response.data[2];
        this.type3Goods = response.data[3];
      })
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + 'px';
  },
  methods: {
    addOrderList(goods) {
      let isHave = false;
      //1.判断点击的商品是否已经存在在订单列表里
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId == goods.goodsId) {
          isHave = true;//存在
        }
      }
      //2.根据isHave的值判断订单列表里是否有这个商品
      if (isHave) {
        //如果存在，增加数量
        let arr = this.tableData.filter(item => item.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        //如果不存在，将goods对象传入订单列表数组
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        }
        this.tableData.push(newGoods);
        this.getCount();
      }
    },
    //删除单个商品
    deleteGoods(goods) {
      this.tableData = this.tableData.filter(item => item.goodsId != goods.goodsId);
      this.getCount();
    },
    delAll() {
      this.totalMoney = 0;
      this.totalCount = 0;
      this.tableData = [];
    },
    checkOut() {
      if (this.totalCount != 0) {
        this.totalCount = 0;
        this.totalMoney = 0;
        this.tableData = [];
        this.$message({
          message: '结账成功，恭喜你又有提成拿了',
          type: 'success'
        });
      } else {
        this.$message.error('憋摁了，咱妹买东西啊')
      }
    },
    getCount() {
      this.totalMoney = 0;
      this.totalCount = 0;
      //进行数量和价格的汇总计算
      if (this.tableData) {
        this.tableData.forEach((element) => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + (element.count * element.price);
        });
      }

    }
  }
}
</script>
<style scoped>
.clearfix {
  clear: both;
}

.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
}

.divBtn {
  margin-top: 10px;
}

.title {
  height: 40px;
  border-bottom: 1px solid #d3dce6;
  line-height: 40px;
  background-color: #f9fafc;
  text-align: left;
}

.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 10px;
  border-radius: 4px;
  background-color: #fff;
}

.often-goods-list ul li .o-price {
  color: #58b7ff;
}

.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #E5E9F2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
}

.cookList li span {

  display: block;
  float: left;
}

.foodImg {
  width: 40%;
}

.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}

.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}

.total {
  background: #fff;
  padding: 10px;
  border-bottom: 1px solid #030303;
}
</style>
