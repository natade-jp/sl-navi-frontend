// emitなし
// mountedなし
<template>
  <div id="shops" class="col shops">
    <div class="shop_top">施設</div>
          <div v-for="z in shops" :id="z|shop_id" :key="z.flag" class="f" tabindex="0">
                <!-- 看板と人数 -->
                <img class="flag" :src="z|flag_img"  :class="z|event_class" >
                <div class="memo">
                    <span v-if="existStaff(z)" class="sn">{{ z|staff_num }}</span>
                    <span v-if="existGuest(z)" class="cn">{{ z|guest_num }}</span>
                    <span v-if="z.female>0" class="female">
                        ♀×{{ z.female }}
                    </span>
                    <span v-if="z.male>0" class="male">
                        ♂×{{ z.male }}
                    </span>
                    <span v-if="isShopClose(z)" class="badge badge-secondary">閉店中</span>
                </div>
                <!-- ポップアップの中身 -->
                <b-popover triggers="click blur" placement="bottom" style="display:none"
                            :target="z|shop_id"
                            :show="shops.length===1"
                >
                    <template v-slot:title>
                        <a :href="z|search_url" v-html="z.name"></a>
                        <b-badge v-if="isEvent(z)" variant="danger">
                            イベント中
                        </b-badge>
                    </template>

                    <span v-html="shop_description(z)"></span><br>
                    <span v-if="isAdultShop(z)" class="badge badge-danger">アダルト施設</span>
                    <span v-if="isKenzenShop(z)" class="badge badge-primary">一般施設</span>
                    <span v-for="tag in z.tags" :key="tag.id" class="badge badge-light">{{ tag }}</span><br>
                    <span class="n2">
                        スタッフ<span class="sn2">{{ z.sn }}</span>人
                        <span v-if="z.staffs.length!=0">
                            (<a v-for="staff in z.staffs" :href="staff|staff_url" :key="staff.id"
                                target=_blank  :class="staff|staff_sex">
                                <template v-if="staff!=''">
                                    {{ staff|staff_name }}
                                </template>
                            </a>)
                        </span>
                        / 訪問者<span class="cn2">{{ z|guest_num }}</span>人
                      <template v-if="z.female+z.male>0">
                        / 男女内訳
                        <span v-if="z.female>0">
                            ♀{{ z.female }}人 
                        </span>
                        <span v-if="z.male>0">
                            ♂{{ z.male }}人
                        </span>
                      </template>
                    </span><br>
                    <a target=_blank v-bind:href="z|map_url">
                        <b-icon-map scale="0.8"></b-icon-map>ここに移動({{ z|sim }})
                    </a>
                    <span v-if="existBlog(z)">
                        / <a target="_blank" v-bind:href="z|blog_url">
                            <b-icon-link scale="0.8"></b-icon-link>Web( {{ z|blog_url_short }} )
                        </a>
                    </span>
                    <br>
                    <img class="heatmap" v-bind:src="z|heatmap">
                </b-popover>
          </div>
      </div>
</template>
<script>
import Vue from 'vue'
import axios from 'axios'
Vue.prototype.$axios = axios


var SHOP_SOURCE = '//sl-navi.com/api/shop'
export default {
  name: 'ShopList',
  props: ['mode'],
  data: function () {
    return {
      shops: []
    }
  },
  filters: {
    // shopの中身(z)はz.staffsとz.tags以外は直接使わず全部フィルタで表示
    'shop_id': function (z) {
      return z.flag
    },
    'event_class': function (z) {
      if(z.event) {
        return 'now_event'
      }else {
        return
      }
    },
    'event_variant': function (z) {
      if(z.event) {
        return 'danger'
      }else {
        return
      }
    },
    'search_url': function (z) {
      return "#/search/"+z.flag
    },
    'flag_img': function (z) {
      return 'https://secondlife.com/app/image/' + z.flag + '/1'
    },
    'staff_num': function (z) {
      return z.sn
    },
    'guest_num': function (z) {
      return z.cn
    },
    'map_url': function (z) {
      return 'https://maps.secondlife.com/secondlife/' + z.sim + '/' + z.x + '/' + z.y + '/' + z.z
    },
    'sim': function (z) {
      return z.sim
    },
    'blog_url': function (z) {
      return z.blog
    },
    'heatmap': function (z) {
      return '/static/heatmap/' + z.flag + '.png?'
    },
    'blog_url_short': function (z) {
      return z.blog.replace(/^http(|s):\/\//, '')
    },
    'staff_url': function (value) {
      return '/d2p/' + value[0]
    },
    'staff_name': function (value) {
      return value[0]
    },
    'staff_sex': function (value) {
      if (value[1] == 0){
        return "female"
      }else if(value[1] == 1){
        return "male"
      }else{
        return
      }
    }
  },
  methods: {
    isEvent (z) {
      return z.event
    },
    existStaff (z) {
      return (z.sn > 0)
    },
    existGuest (z) {
      return (z.cn > 0)
    },
    isShopClose (z) {
      return (z.status === 0)
    },
    isAdultShop (z) {
      return (z.h === 1)
    },
    isKenzenShop (z) {
      return (z.h === 2)
    },
    shop_description (z) {
      let text=z.info
      const url_pattern=/(https?:\/\/[^ \r\n]+)/g
      return text.replace(url_pattern,'<a target="_blank" href="$1">$1</a>').replace(/\n/g, '<br>')
    },
    existBlog (z) {
      return (z.blog !== 'http://www.google.com' && z.blog.indexOf('http') === 0)
    },
    async getShops (m, tagid, search) {
      // console.log(['shoplist:getShop:axios', tagid, m, search])
      await axios.get(SHOP_SOURCE, {
        params: {
          tagid: tagid,
          mode: m,
          search: search
        }
      }).then(res => {
        this.shops = res.data
        // console.log(['shoplist:getShop:then', this.shops])
      })
    }
  }
}
</script>

<style scoped>
  /* ====  ShopList.vue ==== */

  /* 開店中のお店を全部表示リンク（この機能は消えたので使っていない） */
  .view_open_shop{
    color: #ffaa00;
    border-bottom: outset 1px #ffaa00;
  }
  /* 看板 */
  .f{
    margin:4px;
    position: relative;
    display: inline-block;
    width: 150px;
    height: 150px;
    cursor: pointer;

  }
  /* 看板の上にcn snを並べて乗せるdiv */
  .memo{
    align-items: flex-end;
    position: absolute;
    left: 0px;
    bottom: 0px;
    display: flex;
  }
  .cn {
    width: 25px;
    height: 25px;
    color: #ffffff;
    background-color: #2779bd;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
    cursor: pointer;
  }
  .sn{
    width: 25px;
    height: 25px;
    color: #ffffff;
    background-color: #ffaa00;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
    cursor: pointer;
  }
  .male{
    height: fit-content;
    font-size: 60%;
    color: #ffffff;
    border-radius: 25%;
    cursor: pointer;
    background-color: #00aaaa;
  }
  .female{
    height: fit-content;
    font-size: 60%;
    color: #ffffff;
    border-radius: 25%;
    cursor: pointer;
    background-color: #ff8070;
  }


  .n2{
    font-size: 75%;
    font-weight: 700;

  }
  .cn2{
    width: 25px;
    height: 25px;
    padding: 0px 4px;
    color: #ffffff;
    background-color: #2779bd;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
  }
  .sn2{
    width: 25px;
    height: 25px;
    padding: 0px 4px;
    color: #ffffff;
    background-color: #ffaa00;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
  }
  /* ポップオーバー系 */
  .popover {
    max-width: 700px;
  }

  .popover-header{
    background: #fdf0e0;
    border-bottom: dashed 2px #ffb03f;
    padding: 0.3em 0.5em 0.3em 0.5em;
  }

  .popover {
    border: outset 1px #ffb03f;
    border-radius: 9px;
  }
  .flag {
    width: 150px;
    height: 150px;
  }
  .now_event{
    border: solid 3px #ff0000;
  }

  .heatmap {
    height: 150px;
    width: 400px;
  }
    .shops{
    background: #fdfcec;
    border: 1px solid #ffb03f;
    border-radius: 5px;
    padding: 0;
    margin-left: 15px;
    margin-bottom: 15px;
  }
  .shop_top{
    font-size: medium;
    background: #ffc107;
    color:#666666;
    font-weight: bold;
    padding: 0 0 0.2em 0.7em;
    list-style-type: none!important;
  }
</style>
