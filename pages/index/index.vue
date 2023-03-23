<template>
  <view class="page h-full-screen text-base">
    <img src="/static/bg.png" class="ikon" mode="widthFix" alt="">
    <view class="card">
      <div class="card-group">
        <div class="tit">
          <span class="mark"></span>
          <span class="pl-1">设备列表</span>
        </div>
        <div class="btn" @click="startBluetoothDevicesDiscovery">搜索设备</div>
      </div>

      <div class="derives">
        <div class="empty text-sm" v-show="derives.length === 0">
          <div v-if="errMsg">{{ errMsg }}</div>
          <div v-else>未发现设备, 请进入设备 CFG Mode 后重试</div>
        </div>
        <div class="item" v-for="d in derives" :key="d.deviceId">
          <div>{{ d.name }}</div>
          <div class="btn" @click="handleToConfig(d)">配置</div>
        </div>
      </div>
    </view>

    <view class="card">
      <div class="tit">
        <span class="mark"></span>
        <span>使用须知</span>
      </div>
      <div class="text-sm" style="padding-top: 10px;">
        <p>1. 需开启手机蓝牙，部分安卓设备需要授予微信定位权限才可搜索到设备</p>
        <p>2. 长按配置按钮进入配网模式</p>
        <p>3. 具备可用的 2.4G 无线网</p>
      </div>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      derives: [],
      inDiscovery: false,
      errMsg: null,
    }
  },

  onLoad() {
    this.onBluetoothDeviceFound()
    this.openBluetoothAdapter()
  },

  methods: {
    handleToConfig(device) {
      uni.navigateTo({
        url: `/pages/config/index?id=${device.deviceId}&name=${encodeURIComponent(device.name)}`
      })
    },

    stopBluetoothDevicesDiscovery() {
      this.inDiscovery && uni.stopBluetoothDevicesDiscovery()
    },

    onBluetoothDeviceFound() {
      uni.onBluetoothDeviceFound(res => {
        res.devices.forEach(({ name, localName, deviceId }) => {
          if (name?.startsWith('OpenBlock') || localName?.startsWith('OpenBlock')) {
            let index = this.derives.findIndex(d => d.deviceId === deviceId)
            index == -1 && this.derives.push({ name: name || localName, deviceId: deviceId })
          }
        })
      })
    },

    startBluetoothDevicesDiscovery() {
      uni.showLoading({ title: '搜索设备中', mask: true })
      this.inDiscovery = true
      // 开始搜寻附近的蓝牙外围设备
      uni.startBluetoothDevicesDiscovery({
        allowDuplicatesKey: false,
      })
      
      
      setTimeout(_ => {
        this.stopBluetoothDevicesDiscovery()
        uni.hideLoading()
        this.inDiscovery = false
      }, 1000 * 5)
    },

    openBluetoothAdapter() {
      uni.openBluetoothAdapter({
        mode: 'central',
        success: res => {
          this.startBluetoothDevicesDiscovery()
        },
        fail: res => {
          return console.log(res)
          if (res.errCode !== 10001) {
            this.errMsg = res.errMsg
            return uni.showToast({
              icon: 'none',
              title: res.errMsg
            })
          }

          uni.onBluetoothAdapterStateChange((res) => {
            if (!res.available) {
              return uni.showToast({
                icon: 'none',
                title: '蓝牙状态不可用'
              })
            }

            this.startBluetoothDevicesDiscovery()
          })
        }
      })

    },
  }
}
</script>

<style lang="scss">
.ikon {
  width: 100vw;
  height: auto;
}
</style>
