<template>
  <div id="app">
    <keep-alive>
      <router-view/>      
    </keep-alive>
  </div>
</template>

<script>
export default {
  name:'app',
  created(){
    this.getLocation();
  },
  methods:{
    getLocation() {
      // 移动端
      // 移动端包括手机，pad和其它带有GPS定位芯片的智能设备（如手表、音箱等），移动端的系统包括iOS和Android。成功完成定位需要达成以下前提条件：

      // （1）系统GPS打开
      // （2）所使用的App或浏览器已获取定位权限
      // （3）对打开的页面允许使用定位
      // （4）对于iOS10以上系统和Android的一些版本已禁止在非HTTPS协议的域名下定位，请尽快将站点升级到HTTPS
      // 注意，以上只是定位成功的前提条件，满足这些并不一定等于可以成功定位，定位还与当前位置（室内会影响GPS信息）、手机信号和定位权限等因素影响。如果您在使用过程中定位失败，可以参考FAQ：Geolocation的定位流程以及定位失败的原因 ，将失败信息通过工单发送给我们，高德的工程师将协助您解决问题。
      // PC
      // 因为pc设备上大都缺少GPS芯片，所以在PC上的定位主要通过IP精准定位服务，该服务的失败率在5%左右。所以我们用反定位处理，这个定位为非精准定位，但是只是很少的人会遇到这种情况
      const self = this;
      AMap.plugin('AMap.Geolocation', function() {
        var geolocation = new AMap.Geolocation({
          // 是否使用高精度定位，默认：true
          enableHighAccuracy: true,
          // 设置定位超时时间，默认：无穷大
          timeout: 10000,
          // 定位按钮的停靠位置的偏移量，默认：Pixel(10, 20)
          // buttonOffset: new AMap.Pixel(10, 20),
          //  定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
          // zoomToAccuracy: true,     
          //  定位按钮的排放位置,  RB表示右下
          // buttonPosition: 'RB'
        })

        geolocation.getCurrentPosition()
        AMap.event.addListener(geolocation, 'complete', onComplete)
        AMap.event.addListener(geolocation, 'error', onError)

        function onComplete (data) {
          // data是具体的定位信息
          // console.log(data);
          self.$store.dispatch("setLocation",data);
          self.$store.dispatch("setAddress",data.formattedAddress);


        }

        function onError (data) {
          // 定位出错
          //console.log(data);
          self.getLngLatLocation();
        }
      })
    },
    getLngLatLocation(){
      const self = this;
      //根据经纬度反定位到地址
      AMap.plugin('AMap.CitySearch', function () {
        var citySearch = new AMap.CitySearch()
        citySearch.getLocalCity(function (status, result) {
          if (status === 'complete' && result.info === 'OK') {
            // 查询成功，result即为当前所在城市信息
            console.log(result);
            AMap.plugin('AMap.Geocoder', function() {
            var geocoder = new AMap.Geocoder({
              // city 指定进行编码查询的城市，支持传入城市名、adcode 和 citycode
              city:result.adcode
            })
          
            var lnglat =  result.rectangle.aplit(';')[0].split(",");

            geocoder.getAddress(lnglat, function(status, data) {
              if (status === 'complete' && data.info === 'OK') {
                  // result为对应的地理位置详细信息
                  //console.log(data);
                  self.$store.dispatch("setLocation",{
                    addressComponent: {
                      city: result.city,
                      province: result.province
                    },
                    formattedAddress: data.regeocode.formattedAddress
                  });
                  self.$store.dispatch("setAddress",data.regeocode.formattedAddress);
              }
            })
          })
          }
        })
      })
    }
  }
}
</script>  
<style>
#app{
  width:100%;
  height:100%;
  font-size: 14px;
  background: #f1f1f1;
}
</style>
