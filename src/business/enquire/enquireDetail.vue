<template lang="html">
  <div>
    <v-head isFixed="true" goBack="true">
      <div slot="center-part">询价详情</div>
    </v-head>
    <div v-if="detail">
    <div class="enquire-tips" v-if="detail.status!='fail'">
      <tips>
        <p slot="text" v-if="detail.status=='enquiring'">您的询价单已成功提交，请耐心等待。</p>
        <p slot="text" v-if="detail.status=='enquired'">场地方已报价，请尽快下单，以避免耽误排期！</p>
        <p slot="text" v-if="detail.status=='cancel'">您的询价单已取消!</p>
      </tips>
    </div>
    <div class="field-info">
      <div class="title">场地信息</div>
      <div class="content">
        <div class="item"><span class="name">场地名称：</span><span class="txt ellipsis">{{ detail.name }}</span></div>
        <div class="item">
          <span class="name">场地价格：</span>
          <span class="price-num ellipsis" v-if="detail.status=='enquiring'">询价中</span>
          <span class="price-num ellipsis" v-else-if="detail.status=='enquired'">¥{{ detail.price - detail.subsidy }}
          <!--<del class="del">¥{{ detail.price - 0 }}</del>-->
          </span>
          <span class="price-num ellipsis" v-else-if="detail.status=='fail'">询价失败</span>
          <span class="price-num" v-else>已取消</span>
        </div>
        <div v-if="detail.status=='enquired'" :style="{ marginTop : '0.2rem' }">
          <div class="item" v-if="detail.deposit>0"><span class="name">押金：</span><span class="price-num ellipsis">¥{{ detail.deposit }}</span></div>
          <div class="item" v-if="detail.service_fee>0"><span class="name">服务费：</span><span class="price-num ellipsis">¥{{ detail.service_fee }}</span></div>
        </div>

      </div>
      <div class="btns" v-if="detail.status=='enquiring'">
        <span class="silver-btn" @click.stop="queryModel=true">取消询价</span>
        <span class="silver-btn" @click.stop="showServer">联系客服</span>
      </div>
      <div class="btns" v-else-if="detail.status=='enquired'">
        <router-link v-if="detail.field_order_item_id" :to="{ name : detail.order_status ==='submitted' ? 'submittedOrderDetail' : 'bigOrderDetail' , params:{ id : detail.field_order_id } }" class="buy-btn">查看订单</router-link>
        <span v-else-if="!detail.field_order_id && !detail.expired" class="buy-btn" @click.stop="orderNow(detail)">立即下单</span>
        <span v-else-if="!detail.field_order_id && detail.expired" class="buy-btn not">立即下单</span>
        <span v-else></span>
      </div>
      <div class="btns fail" v-else-if="detail.status=='fail'">
        <span class="fail-tit">失败原因：</span>
        <p class="txt">{{ detail.reason }}</p>
        <router-link to="/demand" class="demand-btn">去发布需求</router-link>
      </div>
      <div v-else></div>
    </div>

    <div class="act-info">
      <div class="title">活动信息</div>
      <div class="info-box">
        <div class="item"><span class="name">公司名称：</span><span class="txt ellipsis">{{ detail.company }}</span></div>
        <div class="item"><span class="name">产品名称：</span><span class="txt ellipsis">{{ detail.product }}</span></div>
        <div class="item"><span class="name">推广形式：</span><span class="txt ellipsis">{{ detail.spread_way }}</span></div>
        <div class="item"><span class="name">活动时间：</span><span class="txt ellipsis">{{ detail.start | timeFormat }} - {{ detail.end | timeFormat }}</span></div>
        <div class="item"><span class="name">所需面积：</span><span class="txt ellipsis">{{ detail.size }}</span></div>
        <div class="item" v-if="detail.filename">
          <span class="name">活动方案：</span><a :href="detail.plan_url" class="file-link">{{ detail.filename }}</a>
        </div>
        <div class="act-case" v-else>
          <span class="name">活动方案：</span>
          <div class="img-box">
            <img v-lazy="item.src" v-for="(item , index) in imgList" alt=" " class="img-item" @click.stop="showImg(index)" :key="index">

          </div>
        </div>
      </div>
    </div>

    <div class="links" v-if="detail.status=='fail'">
      <div class="item"><span class="name">联系人：</span><span class="txt ellipsis">{{ detail.user_name }}</span></div>
      <div class="item"><span class="name">联系方式：</span><span class="txt ellipsis">{{ detail.mobile }}</span></div>
    </div>
    <div class="links" v-else>
      <div class="item"><span class="name">发布时间：</span><span class="silver ellipsis">{{ detail.created }}</span></div>
    </div>

    </div>

    <previewer :list="imgList" ref="previewer"></previewer>
    <div v-transfer-dom>
      <confirm
        v-model="queryModel"
        :close-on-confirm="false"
        title="是否取消询价?"
        @on-confirm="onConfirm">
      </confirm>
    </div>
  </div>
</template>

<script>
  import vHead from 'src/components/vHead.vue';
  import tips from 'src/components/tips.vue' ;
  import { Previewer , Confirm, TransferDomDirective as TransferDom } from 'vux';
  import { getEnquireDetail , deleteEnquire } from 'src/service/getData';
  import { timeFormat } from 'src/filters/index';
  import { setSession } from 'src/config/tools';
export default {
  directives: {
    TransferDom
  },
  data () {
    return {
      id : this.$route.params.id ,
      imgList : [],
      detail: null ,
      queryModel : false ,
    }
  },
  filters:{ timeFormat },
  components:{
    vHead , tips , Previewer , Confirm
  },
  methods:{
    init(){
      getEnquireDetail( this.id ).then(res => {
        this.detail = res.result ;
        let imgArr = res.result.enquiry_information_images ;
        this.imgList = imgArr.map( ( item ) => {
          return {
            src : item
          }
        });
        this.$vux.loading.hide()
      }).catch(err => { this.$vux.loading.hide() })
    },
    showImg(n){
      this.$refs.previewer.show(n)
    },
    onConfirm(){
      deleteEnquire( this.id ).then(res => {
         this.$router.push('/enquirelist');
      }).catch(err => {
        this.$vux.toast.show({
          type : 'warn',
          text: '失败'
        })
      })
    },
    showServer(){
      _MEIQIA('showPanel');
    },
    getDate(a,b){
      let start = new Date( a ).getTime();
      let end   = new Date( b ).getTime() ;
      let temp  = (end - start)/(1*24*3600*1000) + 1;
      return temp ;
    },
    orderNow( detail ){
      if( detail.expired ) return ;
//      console.log( detail );

      let arr = [{
        enquiry_id : +this.id ,
        id : +detail.sid ,
        size : detail.size ,
        lease_term_type_id : 1 ,
        count : 1,
        count_of_time_unit : this.getDate( detail.start , detail.end ) ,
        custom_dimension : '',
        date : detail.start ,
        end : detail.end,
        lease_term_type_name : '天',
        name : detail.name,
        pic_url : detail.pic_url,
        discount_rate : 0 , // 补贴率
        discount_price : detail.subsidy*100 , // 补贴钱
        field_type : detail.field_type ,
        price : detail.price*100 - detail.subsidy*100, // 价格
        deposit : detail.deposit,
        leaveWorld : '', // 留言
        service_fee : detail.service_fee // 服务费
      }];
//      console.log(arr);
      setSession('orderResource',{ resource : arr , cart_item_ids : null });
       this.$router.push('/submitOrder');
    }
  },
  created(){
    this.$vux.loading.show({ text: 'loading' });
  },
  mounted(){
    this.init();
    _MEIQIA('init');
  }
}

</script>

<style lang="scss" scoped>
  @import "../../style/mixin";
  .title{
    line-height: 0.9rem;
    padding: 0 0.24rem;
    @include sc(0.3rem,$fontColor);
  }
  .field-info{
    background-color: #fff;
    .content{
      padding: 0.3rem 0.24rem;
      border-top: 1px solid #e8e8e8;
      border-bottom: 1px solid #e8e8e8;
      font-size: 0.28rem;
      .item{
        display: flex;
        .name{
          width: 1.42rem;
          color: #999;
        }
        .txt{
          color: #000;
        }
        .price-num{
          color: $red;
          .del{
            @include sc(0.22rem,#999);
          }
        }
      }
      .item + div.item{
        margin-top: 0.2rem;
      }
    }
    .btns{
      padding: 0.3rem 0.24rem;
      text-align: right;
      .silver-btn{
        display: inline-block;
        width: 1.72rem;
        height: 0.52rem;
        margin-left:0.4rem;
        line-height: 0.52rem;
        text-align: center;
        border-radius: 0.04rem;
        border: 1px solid #D8D8D8;
        @include sc(0.28rem,#666);
      }
      .buy-btn{
        display: inline-block;
        width: 1.72rem;
        height: 0.52rem;
        margin-left:0.4rem;
        line-height: 0.52rem;
        text-align: center;
        border-radius: 0.04rem;
        border: 1px solid #FF6600;
        @include sc(0.28rem,#FF6600);
        &.not{
          border: 1px solid #D8D8D8;
          @include sc(0.28rem,#999);
        }
      }
      .demand-btn{
        width: 1.78rem;
        height: 0.52rem;
        line-height: 0.52rem;
        text-align: center;
        border-radius: 0.04rem;
        @include sc(0.28rem,$mainColor);
        border: 1px solid $mainColor;
        background-color: #fff;
      }
      &.fail{
        display: flex;
        align-items: center;
        text-align: left;
        .fail-tit{
          @include sc(0.24rem,$fontColor);
          font-weight: bold;
        }
        .txt{
          flex: 1;
          width: 100%;
          @include sc(0.24rem,$fontColor);
          font-weight: bold;
        }
      }
    }
  }
  .act-info{
    margin-top: 0.2rem;
    background-color: #fff;
    .info-box{
      padding: 0.3rem 0.24rem;
      border-top: 1px solid #e8e8e8;
      font-size: 0.28rem;
      .item{
        display: flex;
        .name{
          width: 1.42rem;
          color: #999;
        }
        .txt{
          color: #000;
        }
        a.file-link{
          color: $mainColor;
          text-decoration: underline;
        }

      }
      .item + div.item{
        margin-top: 0.2rem;
      }
      .act-case{
        display: flex;
        .name{
          margin-top: 0.2rem;
          color: #999;
        }
        .img-box{
          display: flex;
          flex-wrap: wrap;
          width: 5.55rem;
          .img-item{
            width: 1.5rem;
            height: 1.5rem;
            margin: 0.2rem 0.2rem 0 0;
            border-radius: 0.08rem;
          }
        }
      }
    }
  }
  .links{
    margin-top: 0.2rem;
    padding: 0.3rem 0.24rem;
    font-size: 0.28rem;
    background-color: #fff;
    .item{
      display: flex;
      .name{
        width: 1.42rem;
        color: #999;
      }
      .txt{
        color: #000;
      }
      .silver{
        color: #999;
      }
    }
    .item + div.item{
      margin-top: 0.2rem;
    }
  }
</style>
