<template lang="html">
<!--场地列表单个组件-->
  <div class="v-field-list-item" v-show="field">
    <router-link :to="{ name : 'fieldsDetail' , params : { id : field.resource_id } , query : { res_type_id : field.res_type_id } }" class="item-wraper">
      <div class="img-box">
        <img v-lazy="field.pic_url" :data-srcset="field.pic_url | half" :alt="field.resource_name">
        <v-label :labelText="field.activity_type" v-if="field.res_type_id==3"></v-label>
        <v-label :labelText="field.field_type" v-else></v-label>
        <span class="expired-mask" v-if="field.expired">报名已截止</span>
        <span class="city-name" v-if="showCity">{{ field.city.name }}</span>
        <!-- 城市 -->
         <div class="label-icon" v-if="field.res_type_id==3"></div>
        <!--活动  限时-->
        <div class="hot-icon" v-else-if="field.is_hot"></div>
        <!--场地  热卖-->
      </div>
      <div class="field-text-content"  :class="{ expired : field.expired }">
        <div class="title ellipsis">
          <span class="field-name">{{ field.resource_name }}</span>
        </div>
        <div class="info ellipsis" v-if="field.res_type_id==3">
          <span>活动时间：</span><span>{{ field.activity_start_date | timeFormat }}</span><span class="split-line">-</span><span>{{ field.deadline | timeFormat }}</span>
        </div>
        <div class="info ellipsis" v-else>
          <!--<span class="type-label" v-if="field.cooperation_type_id == 12">自营</span>-->
          <span>人流量：</span><span>{{ field.number_of_people || '' }}</span> / <span class="number-people">付款人数：</span><span>{{ field.order_count }}</span>
        </div>

        <div class="address ellipsis">{{field.district}}{{field.address}}</div>
        <!--地铁相关暂时注释-->
        <!--<div class="distance-station" v-if="field.distance">-->
          <!--距{{ field.station }}站{{ field.distance }}公里-->
        <!--</div>-->
        <div class="tags">

          <span class="item" v-if="field.deposit<=0">免押金</span>
          <span class="item" v-if="field.subsidy_str">{{ field.subsidy_str }}</span>
        </div>
        <div class="price-box ellipsis" v-if="field.is_enquiry">
          <span class="face-price">面议</span><span class="refer-price">参考价：¥{{ field.refer_min_price/100 }}~¥{{ field.refer_max_price/100 }}</span>
        </div>
        <div class="price-box ellipsis" v-else>
          <span class="price-label" v-if="field.subsidy_fee">惠</span><span class="price-pre">¥</span><span class="price">{{ (field.price - field.subsidy_fee)  / 100 | twoPointPre }}.</span><span class="price-next">{{ (field.price - field.subsidy_fee)  / 100 | twoPointNext }}</span>
          <span class="com-font">&nbsp;/ {{ field.unit }}起</span>
          <del v-if="field.subsidy_fee" class="com-font oringe-price">¥{{ field.price /100 }}</del>
        </div>


      </div>
    </router-link>
  </div>
</template>

<script>
  import vLabel from 'src/components/vLabel.vue';
  import { half , twoPointPre , twoPointNext , timeFormat } from 'src/filters';

export default {
  data () {
    return {

    }
  },
  props : [ 'field' , 'showCity' ],
  components : { vLabel } ,
  filters : { half , twoPointPre , twoPointNext , timeFormat }
}

</script>

<style lang="scss" scoped>
  @import "../style/mixin";
  .com-font{
    font-size: 0.22rem;
    color: #999;
  }
  .v-field-list-item{
    padding: 0.3rem 0.24rem;
    border-bottom: 1px solid #eee;
    background-color: #fff;
    .item-wraper{
      display: flex;
      justify-content: space-between;
      .img-box{
        position: relative;
        @include wh(2rem,2rem);
        > img{
          @include wh(2rem,2rem);
        }
        .city-name{
          position: absolute;
          top: 0.1rem;
          left: 0;
          width: auto;
          height: 0.3rem;
          padding: 0 0.1rem;
          font-size: 0.24rem;
          color: #fff;
          background-color: $yellow;
        }
        .expired-mask{
          @include center;
          @include wh( 1.4rem , 1.4rem );
          line-height: 1.4rem;
          text-align: center;
          border-radius: 50%;
          font-size: 0.24rem;
          color: #fff;
          background-color: rgba(0,0,0,0.6);
        }
        .label-icon{
          position: absolute;
          top: 0.08rem;
          right: 0.11rem;
          @include wh(0.8rem,0.42rem);
          @include bg-image('../images/ic_xianshi');
        }
        .hot-icon{
          position: absolute;
          top: 0.08rem;
          right: 0.11rem;
          @include wh(0.8rem,0.42rem);
          @include bg-image('../images/ic_remai');
        }
      }
      .field-text-content{
        padding-top: 0.05rem;
        width: 4.74rem;
        font-size: 0;
        /*overflow: hidden;*/
        .title{
          @include sc(0.3rem,$fontColor);
        }
        .address{
          margin-top: 0.2rem;
          @extend .com-font ;
        }
        .distance-station{
          margin-top: 0.2rem;
          @extend .com-font ;
        }
        .info{
          margin-top: 0.2rem;
          @extend .com-font ;
          .split-line{
            margin: 0 0.05rem;
          }
          .type-label{
            display: inline-block;
            vertical-align: top;
            @include wh( 0.52rem , 0.26rem );
            border: 1px solid $orange;
            border-radius: 0.02rem;
            margin-right: 0.1rem;
            text-align: center;
            line-height: 0.24rem;
            @include sc(0.2rem,$orange);
          }
        }
        .tags{
          display: flex;
          align-items: center;
          .item{
            height: 0.32rem;
            line-height: 0.32rem;
            margin: 0.16rem 0.16rem 0 0;
            padding: 0 0.06rem;
            background-color: rgba(255,102,0,0.1);
            border-radius: 0.02rem;
            @include sc(0.2rem, $orange);
          }
        }
        .price-box{
          margin-top: 0.25rem;
          .face-price{
            margin-right: 0.1rem;
            font-family: PingFangSC-Medium;
            @include sc(0.34rem,$red);
          }
          .refer-price{
            @include sc(0.22rem, $fontColor);
          }
          .price-label{
            display: inline-block;
            vertical-align: text-bottom;
            @include wh( 0.24rem , 0.24rem );
            margin-right: 0.1rem;
            @include bg-image('../images/ic_hui_three_one');
          }
          .price-pre{
            font-size: 0.22rem;
            color: $red;
          }
          .price{
            font-size: 0.36rem;
            color: $red;
            font-weight: bold;
          }
          .price-next{
            @extend .price-pre;
          }
          .oringe-price{
            margin-left: 0.1rem;
          }
        }
        &.expired{
          .title{
            .field-name{
              color: #999;
            }
          }
          .tags{
            .item{
              color: #999;
              background-color: rgba(153,153,153,0.1);
            }
          }
          .price-box{
            .price-label{
              @include bg-image('../images/ic_hui_gary_three_one');
            }
            .price-pre{
              color: #999;
            }
            .price{
              color: #999;
            }
            .price-next{
              @extend .price-pre;
            }
          }
        }
      }
    }
  }

</style>
