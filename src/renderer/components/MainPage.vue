<template>
<div id="mainPage" class="window">
  <div class="window-content">
  <div class="pane-group">
    <div class="pane-sm sidebar">
      <nav class="nav-group">
        <h5 class="nav-group-title">配置</h5>
        <a v-for="(item, index) in items" @click="tab(index)" class="nav-group-item" :class="{active: activeIndex === index} ">
          <span class="icon" :class="item.icon"></span>
          {{item.title}}
        </a>
        <div class="nav-steps">
            <select  v-show="activeIndex == 0"  class = "form-control"  @change="open()"  v-model="selected">
                 <option value="请选择要导入的文件">请选择要导入的文件</option>
                 <option v-for="val in files">{{val}}</option>
            </select>
          <p>进度: {{ configedCount }}/6</p>
          <button @click="myreset" type="button" class="btn btn-form btn-primary">重置</button>
          <button @click="next" type="button" class="btn btn-form btn-disable" :class="{'btn-primary': configedCount >= 6}">下一步</button>
        </div>
      </nav>
    </div>

    <div class="pane">
      <div v-show="activeIndex == 0">
        <general-form @submitFrameConfig="receivedFromClild" name="general" ref="step0"></general-form>
      </div>
      <div v-show="activeIndex == 1">
        <channel-form @submitFrameConfig="receivedFromClild" name="channelA" ref="step1"></channel-form>
      </div>
      <div v-show="activeIndex == 2">
        <channel-form @submitFrameConfig="receivedFromClild" name="channelB" ref="step2"></channel-form>
      </div>
      <div v-show="activeIndex == 3">
        <channel-form @submitFrameConfig="receivedFromClild" name="channelC" ref="step3"></channel-form>
      </div>
      <div v-show="activeIndex == 4">
        <channel-form @submitFrameConfig="receivedFromClild" name="channelD" ref="step4"></channel-form>
      </div>
      <div v-show="activeIndex == 5">
        <combine-form @submitFrameConfig="receivedFromClild" name="combine" ref="step5"></combine-form>
      </div>
    </div>
  </div>
  </div>
</div>
</template>

<script>
  import Device from '@/services/device'
  import Ini from '@/services/ini'
  import SystemInformation from './LandingPage/SystemInformation'
  import GeneralForm from './MainPage/GeneralForm'
  import CombineForm from './MainPage/CombineForm'
  import ChannelForm from './MainPage/ChannelForm'
  export default {
    name: 'main-page',
    components: { SystemInformation, GeneralForm, CombineForm, ChannelForm },
    data () {
          let initdata = {
                 activeIndex: 0,
                 configedCount: 0,
                 files: '',
                 selected: '',
                 items: [
                   {title: '通用', icon: 'icon-publish'},
                   {title: '通道A', icon: 'icon-window'},
                   {title: '通道B', icon: 'icon-window'},
                   {title: '通道C', icon: 'icon-window'},
                   {title: '通道D', icon: 'icon-window'},
                   {title: '组合按键', icon: 'icon-window'}
                 ]
               }
         let storage = window.localStorage
         initdata.selected = storage.getItem('file') ? storage.getItem('file') : '请选择要导入的文件'
         let pathdir='./data'
         if(Ini.CheckExistsSync(pathdir)){
             initdata.files = Ini.getFileList(pathdir)
           }
         return  initdata
    },
    mounted () {
      //在子组件初始化后执行1次
      Device.CObject.Clean()
      Device.manager.reset()
      this.$refs.step0.save();
      this.$refs.step1.save();
      this.$refs.step2.save();
      this.$refs.step3.save();
      this.$refs.step4.save();
      this.$refs.step5.save();
    },
    methods: {
      tab (index) {
         //参数是整数，index是到那个页面，this.activeIndex是当前页面
        this.activeIndex = index
      },
      open () {
        let storage = window.localStorage
        if(this.selected=='请选择要导入的文件'){
              storage.removeItem("file")
              return;
        }
        storage.setItem('file',this.selected)
        //只维护第一大步，第二大步是在第一大步的基础上计算出来的
        let agofile = './USU.ini'
        let curfile=`${this.selected}`
        Ini.fileCopy(curfile,agofile)
        window.location.reload()
      },
      receivedFromClild (msg) {
         // 子组件提交配置信息后会调用
        // msg是从子组件传递过来的一个设备配置对象
        // 更新存储配置信息
        Device.manager.update(msg.name, msg)
        // 记录保存了哪一步的配置
        this.configedCount = Device.manager.configedCount()
        //console.log('receivedFromClild',msg)
        //求出最终的配置结果
        msg.dump()
      },
      myreset () {
              let agofile = './USU.ini'
              Ini.fileCopy('./default.ini',agofile)
              window.localStorage.removeItem("file")
              window.location.reload()
      },
      next () {
        if (this.configedCount >= 6) {
            let channels=Device.CObject.matchres
            let mixkey=Device.CObject.mixkeys
            Device.CObject.All=[]
               for (var val1 of channels) {
                 //兼容数字key没有按顺序赋值时的undefined隐患
                if(val1){
                  for (var val2 of val1) {
                        Device.CObject.All.push(val2)
                 }
             }
            }
      let storage = window.localStorage
      let obj=JSON.parse(storage.getItem('combine'))
      //这里过滤没选中的按钮
      for(var key in mixkey){
        if(obj[key]){
            let curobj = mixkey[key]
            Device.CObject.All.push(curobj)
        }
      }

      let mixall= Device.CObject.All
      mixall.forEach(function(value, index) {
           let rsindex = index+5
           mixall[index].id = rsindex
       })
        //将对象数组转换成json字符串
         let totext=JSON.stringify(Device.CObject.All)
         storage.setItem("alldevice",totext)
         this.$router.push('/grouping')
        }
      }
    }
  }
</script>

<style>
.nav-steps { padding:10px 20px }
.nav-steps p{ text-left:30px; }
</style>