<template>
  <div class="page h-full-screen text-base">
    <div style="padding-top: 20px;">
      <view class="card" style="margin-top: 0;">
        <div class="tit">
          <span class="mark"></span>
          <span>{{ deviceName }} - {{ serviceId ? '已连接' : '连接中' }}</span>
        </div>

        <div class="empty" v-show="errMsg">
          {{ errMsg }}
        </div>
      </view>

      <view class="card">
        <div class="card-group">
          <div class="tit">
            <span class="mark"></span>
            <span class="pl-1">可连接 WiFi </span>
          </div>
          <div class="btn" @click="getWifi">刷新</div>
        </div>

        <div class="derives">
          <div class="empty" v-show="wifi.length === 0">
            正在获取 WiFi
          </div>
          <div class="item" v-for="d in wifi" :key="d.name">
            <div class="wifi-name">
              <div class="signal-icon icon-1" v-if="d.signalStrength < 21"></div>
              <div class="signal-icon icon-2" v-else-if="d.signalStrength < 41"></div>
              <div class="signal-icon icon-3" v-else-if="d.signalStrength < 61"></div>
              <div class="signal-icon icon-4" v-else-if="d.signalStrength < 81"></div>
              <div class="signal-icon icon-5" v-else-if="d.signalStrength < 101"></div>
              <div>{{ d.name }}</div>
            </div>
            <div class="btn" @click="handleToConfig(d)">连接</div>
          </div>
        </div>
      </view>
    </div>
  </div>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
      deviceName: null,
      deviceId: null,
      serviceId: null,
      writeCharacteristicId: null,
      readCharacteristicId: null,
      notifyCharacteristicId: null,
      wifi: [],
      wifiPack: '',
      errMsg: null,
      lastCommand: null,
    }
  },

  onLoad(option) {
    this.deviceId = option.id
    this.deviceName = decodeURIComponent(option.name)
    this.deviceId && this.connectToDevice()
  },

  onUnload() {
    wx.closeBLEConnection({
      deviceId: this.deviceName,
    })
  },

  methods: {
    handleToConfig(wifi) {
      uni.showModal({
        title: 'WiFi 密码',
        placeholderText: `WiFi ${wifi.name}的密码`,
        editable: true,
        success: res => {
          if (res.confirm && res.content) {
            if (res.content.length >= 8) {
              uni.showLoading({
                title: '设备尝试连接 WiFi',
                mask: true
              })
              return this.sendDataToDevice(`#2${wifi.name}:${res.content}`)
            }

            uni.showToast({
              title: '密码长度不符合',
              icon: 'none'
            })
          }
        }
      })
    },

    getWifi() {
      this.wifi = []
      this.wifiPack = ''

      uni.showToast({
        title: '获取中',
        icon: 'loading',
        duration: 3000
      })

      this.sendDataToDevice('#1')

      setTimeout(_ => {
        let wifi = this.wifiPack.split('\n')
        this.wifi = wifi.map(w => {
          let [ssid, signalStrength] = w.split(':')
          return {
            name: ssid,
            signalStrength: parseInt(signalStrength || 0)
          }
        })
            .filter(w => w.signalStrength)
            .sort((a, b) => b.signalStrength - a.signalStrength)
      }, 3000)
    },

    sendDataToDevice(data) {
      const buffer = new ArrayBuffer(data.length)
      const dataView = new DataView(buffer)

      for (var i = 0; i < data.length; i++) {
        dataView.setUint8(i, data.charAt(i).charCodeAt())
      }

      this.lastCommand = data

      wx.writeBLECharacteristicValue({
        deviceId: this.deviceId,
        serviceId: this.serviceId,
        characteristicId: this.writeCharacteristicId,
        value: buffer,
        success: res => {
          console.log('send success')
        },
        fail: err => {
          console.log('send error', err)
        }
      })
    },

    onBLECharacteristicValueChange() {
      wx.notifyBLECharacteristicValueChange({
        deviceId: this.deviceId,
        serviceId: this.serviceId,
        characteristicId: this.notifyCharacteristicId,
        state: true,
      })

      // 监听特征值变化
      wx.onBLECharacteristicValueChange(res => {
        let msg = String.fromCharCode.apply(null, new Uint8Array(res.value))
        console.log(`command: ${this.lastCommand}`)
        console.log('resp', msg)

        if (this.lastCommand === '#1') {
          this.wifiPack = `${this.wifiPack}${msg}`
          return
        }

        if (this.lastCommand.startsWith('#2')) {
          uni.hideLoading()
          if (msg === '#5') {
            uni.showModal({
              title: '连接失败',
              content: '连接失败, 请核对密码是否正确',
              showCancel: false
            })
          }

          if (msg === '#4') {
            uni.showModal({
              title: '连接成功',
              content: '设备连接 WiFi 成功',
              showCancel: false
            })
          }
        }
      })
    },

    getBLEDeviceCharacteristics() {
      wx.getBLEDeviceCharacteristics({
        deviceId: this.deviceId,
        serviceId: this.serviceId,
        success: res => {
          res.characteristics.forEach(c => {
            let {
              write,
              notify,
              read
            } = c.properties
            write && notify && (this.notifyCharacteristicId = c.uuid)
            write && !notify && (this.writeCharacteristicId = c.uuid)
            notify && read && (this.readCharacteristicId = c.uuid)
          })

          // 特征值变化监听
          this.onBLECharacteristicValueChange()
          uni.hideLoading()

          setTimeout(_ => {
            // 获取一次 WiFi
            this.getWifi()
          }, 500)

        },
        fail: err => {
          this.errMsg = err.errMsg
          console.log('getBLEDeviceCharacteristics', err)
        }
      })
    },

    getBLEDeviceServices() {
      wx.getBLEDeviceServices({
        deviceId: this.deviceId,
        success: res => {
          this.serviceId = res.services.find(s => s.isPrimary).uuid
          this.getBLEDeviceCharacteristics()
        },

        fail: err => {
          this.errMsg = err.errMsg
          console.log('getBLEDeviceServices', err)
        }
      })
    },

    connectToDevice() {
      uni.showLoading({
        title: '设备连接中'
      })

      wx.createBLEConnection({
        deviceId: this.deviceId,
        success: () => {
          this.getBLEDeviceServices()
        },
        fail: err => {
          console.log('createBLEConnection', err)
          this.errMsg = err.errMsg
        }
      })
    }
  }
}
</script>

<style scoped lang="scss">
.wifi-name {
  display: flex;
  justify-content: flex-start;
  align-items: center;
}

.signal-icon {
  // 73 60
  width: 18px;
  height: 12px;
  background-repeat: no-repeat;
  background-size: 100% 100%;
  margin-right: 5px;
}

.icon-1 {
  background-image: url('../../static/1.png');
}

.icon-2 {
  background-image: url('../../static/2.png');
}

.icon-3 {
  background-image: url('../../static/3.png');
}

.icon-4 {
  background-image: url('../../static/4.png');
}

.icon-5 {
  background-image: url('../../static/5.png');
}
</style>
