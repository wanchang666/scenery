<template>
  <view class="select-location">
    <map id="map" :latitude="selectedSearch.location.lat || latitude"
         :longitude="selectedSearch.location.lng || longitude"
         :markers="markers"
         :scale="16" :show-location="true"
         @callouttap="goToRoutePlan"
         @markertap="goToRoutePlan"
         @regionchange="changeEvent">
      <!-- 顶部搜索框 -->

      <!-- 定位标志 -->
      <cover-image class="location-icon"
        src="/static/images/location.png">
      </cover-image>
      <!-- tip -->
<!--      <cover-view class="tip">您可以拖动地图，标记当前精准位置</cover-view>-->
    </map>
    <view class="search-area">
      <view class="top-search">
        <van-search :value="keyword" show-action
                    background="rgba(255, 255, 255, 0)"
                    shape="round"
                    readonly
                    placeholder="搜索地点"
                    @click.native="goSearch">
        </van-search>
      </view>
      <view class="search-list">
        <scroll-view class="list"
          scroll-y>
          <view :class="['list-item', 'flex', 'van-hairline--bottom', active === item.id ? 'active' : '']"
            v-for="(item) in list"
            :key="item.id"
            @tap="selectedAddress(item)">
            <van-icon v-if="active === item.id"
              class="icon"
              name="checked"
              size="34rpx"
              :color="active === item.id ? '#007aff' : ''">
            </van-icon>
            <view class="info">
              <view class="title ellipsis">{{ item.title }}</view>
              <view class="address ellipsis">{{ item.address }}</view>
            </view>
          </view>
        </scroll-view>
<!--        <van-button type="primary"-->
<!--          block-->
<!--          @click="handleConfirm">-->
<!--          确认位置-->
<!--        </van-button>-->
      </view>
    </view>
  </view>
</template>

<script>
import debounce from 'lodash/debounce'
import { formatDate, reverseGeocoder, getLocation } from '@/utils'
import { mapGetters, mapMutations } from 'vuex'
const chooseLocation = requirePlugin('chooseLocation');
import { MAP_KEY } from 'config'
export default {
  data () {
    return {
      keyword: '',
      active: '',
      longitude: '',
      latitude: '',
      list: [],
      mapContext: null,
      changeEvent: debounce(this.handleRegionChange, 300),
      scenery: []
    }
  },
  computed: {
    ...mapGetters(['selectedLocation', 'selectedSearch']),
    markers(){
      return this.scenery.map(v=>{
        v.iconPath='../../static/images/location-fill.png'
        v.width = 30
        v.height = 30
        v.id = Number(v.rank)
        // 没有id，bindcallouttap不会生效
        v.callout = {
          content: v.name,
          display: 'ALWAYS',
          textAlign: 'center',
          padding: 10,
          borderRadius: 3,
        }
        return v
      })
    }
  },
  onLoad ({ longitude, latitude }) {
    console.log(longitude);
    this.mapContext = uni.createMapContext('map')
    this.getLocation()
    this.getMarkers()
  },
  onShow(){
    // const location = chooseLocation.getLocation();
    const key = MAP_KEY; //使用在腾讯位置服务申请的key
    const referer = '景你所见'; //调用插件的app的名称
    const location = JSON.stringify({
      latitude: 39.89631551,
      longitude: 116.323459711
    });
    const category = '生活服务,娱乐休闲';

    // wx.navigateTo({
    //   url: 'plugin://chooseLocation/index?key=' + key + '&referer=' + referer + '&location=' + location + '&category=' + category
    // });
    console.log(location);
  },
  methods: {
    ...mapMutations(['SET_SELECTED_LOCATION', 'SET_SELECTED_SEARCH']),
    goToRoutePlan({markerId}){
      // console.log(a, b);
      // let plugin = requirePlugin('routePlan');
      let key = MAP_KEY;  //使用在腾讯位置服务申请的key
      const location = this.markers.find(marker => marker.id === markerId)
      console.log(location);
      let referer = '景你所见';   //调用插件的app的名称
      let endPoint = JSON.stringify({  //终点
        'name': location.callout.content,
        'latitude': location.latitude,
        'longitude': location.longitude
      });
      wx.navigateTo({
        url: 'plugin://routePlan/index?key=' + key + '&referer=' + referer + '&endPoint=' + endPoint
      });
    },
    getMarkers(){
      wx.cloud.callFunction({
        name: 'getAllMarkers',
        success: res => {
          this.scenery = res.result.data
          console.log(res);
        },
        fail: err => {
          wx.showToast({
            icon: 'none',
            title: '调用失败',
          })
          console.error('[云函数] 调用失败：', err)
        }
      })

      // const db = wx.cloud.database()
      // db.collection('mapMarkers').where({}).limit(20).get({
      //   success: res => {
      //     console.log(res);
      //   },
      //   fail: err => {
      //     uni.showToast({
      //       icon: 'none',
      //       title: '查询记录失败'
      //     })
      //     console.error('[数据库] [查询记录] 失败：', err)
      //   }
      // })
    },
    addMarker(){
      const db = wx.cloud.database()
      db.collection('test').add({
        data: {
          count: 1
        },
        success: res => {
          // 在返回结果中会包含新创建的记录的 _id
          this.setData({
            counterId: res._id,
            count: 1
          })
          wx.showToast({
            title: '新增记录成功',
          })
          console.log('[数据库] [新增记录] 成功，记录 _id: ', res._id)
        },
        fail: err => {
          wx.showToast({
            icon: 'none',
            title: '新增记录失败'
          })
          console.error('[数据库] [新增记录] 失败：', err)
        }
      })
    },
    dituxuandian(){
      const key = MAP_KEY; //使用在腾讯位置服务申请的key
      const referer = '景你所见'; //调用插件的app的名称
      const location = JSON.stringify({
        latitude: 39.89631551,
        longitude: 116.323459711
      });
      const category = '生活服务,娱乐休闲';

      wx.navigateTo({
        url: 'plugin://chooseLocation/index?key=' + key + '&referer=' + referer + '&location=' + location + '&category=' + category
      });
    },
    goSearch () {
      uni.navigateTo({
        url: `/pages/search/search?longitude=${this.longitude}&latitude=${this.latitude}`
      })
    },
    getLocation () {
      getLocation().then(res => {
          const { longitude, latitude } = res
          this.longitude = longitude
          this.latitude = latitude
          this.getLocationInfo({ longitude, latitude })
        })
        .catch(() => {
          this.location.loading = false
          this.location.error = true
        })
    },

    getLocationInfo (location) {
      reverseGeocoder(location)
        .then(res => {
          let pois = res.result.pois
          if (this.selectedLocation) {
            const index = pois.findIndex(item => item.id === this.selectedLocation.id)
            if (index > -1) pois.splice(index, 1)
            pois.unshift(this.selectedLocation)
          } else if (this.selectedSearch) {
            const index = pois.findIndex(item => item.id === this.selectedSearch.id)
            if (index > -1) pois.splice(index, 1)
            pois.unshift(this.selectedSearch)
          }
          this.list = pois
          if (pois.length) {
            this.active = pois[0].id

          }
        })
    },
    handleRegionChange (e) {
      if (e.type === 'end' && e.causedBy === 'drag') {
        this.SET_SELECTED_LOCATION(null)
        this.SET_SELECTED_SEARCH(null)
        this.mapContext.getCenterLocation({
          success: res => {
            const { longitude, latitude } = res
            this.getLocationInfo({ longitude, latitude })
          }
        })
      }
    },
    selectedAddress (item) {
      console.log(item);
      this.active = item.id
      this.SET_SELECTED_SEARCH(item)
      this.mapContext.moveToLocation({
        latitude: item.location.lat,
        longitude: item.location.lng
      })
    },
    handleConfirm () {
      const index = this.list.findIndex(item => item.id === this.active)
      if (index > -1) {
        this.SET_SELECTED_LOCATION(this.list[index])
        uni.navigateBack()
      }
    }
  },
  watch: {
    selectedSearch (newData) {
      if (newData) {
        const { lng, lat } = newData.location
        this.mapContext.moveToLocation({
          longitude: lng,
          latitude: lat
        })
        this.getLocationInfo({
          longitude: lng,
          latitude: lat
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.select-location {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  #map {
    width: 100%;
    height: 600rpx;
    position: relative;
  }
  /deep/ .top-search {
    .van-search__content {
    }
  }
  .location-icon {
    width: 64rpx;
    height: 64rpx;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  .tip {
    font-size: 20rpx;
    color: $uni-text-color-grey;
    line-height: 42rpx;
    padding: 0 20rpx;
    position: absolute;
    left: 50%;
    bottom: 30rpx;
    box-shadow: 0px 1px 10px 1px rgba(153, 153, 153, 0.34);
    background-color: $uni-bg-color;
    border-radius: 4px;
    transform: translateX(-50%);
  }
  .search-area {
    width: 100%;
    padding: 0 30rpx;
    box-sizing: border-box;
    position: absolute;
    top: 600rpx;
    bottom: 0;
    .search-list {
      height: 100%;
      background-color: $uni-bg-color;
      border-radius: 4px;
      padding: 20rpx 0;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      .list {
        flex: 1;
        overflow: auto;
        .list-item {
          padding: 15rpx 0;
          position: relative;
          &.active {
            .title,
            .address {
              color: $uni-color-primary;
            }
          }
          .info {
            padding-left: 50rpx;
          }
          .icon {
            position: absolute;
            left: 0;
            top: 50%;
            transform: translateY(-50%);
          }
          .title {
            font-size: 28rpx;
            line-height: 40rpx;
          }
          .address {
            font-size: 24rpx;
            height: auto;
            color: $uni-text-color-grey;
          }
        }
      }
    }
  }
}
</style>