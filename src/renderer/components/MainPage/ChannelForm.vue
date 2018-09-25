<template>
  <form>
  	<label>配置: {{name}}</label>
  	<div class="form-group">
  		<label>通道功能</label>
  		<select v-model="type" class="form-control">
        <option value="0">无</option>
  		  <option value="1">开关</option>
  		  <option value="2">开关/调光</option>
  		  <option value="3">窗帘</option>
  		</select>
  	</div>
	  <!-- 开关 -->
	  <template v-if="type == '1'">
	    <div class="form-group">
	      <label>长短按键区分</label>
	      <select v-model="button.distinction" class="form-control">
	        <option value="0">不区分</option>
	        <option value="1">区分</option>
	      </select>
	    </div>
	    <!-- 不区分 -->
	    <template v-if="button.distinction == '0'">
      	<div class="form-group">
      		<label>触点闭合动作（上升沿）</label>
            <select v-model="button.withoutDistinction.r1" class="form-control">
              <option value="0">开启</option>
		      <option value="1">关闭</option>
		      <option value="2">反转</option>
		      <option value="3">无动作</option>
		    </select>
      	</div>
      	<div class="form-group">
      		<label>触点打开动作（下降沿）</label>
            <select v-model="button.withoutDistinction.r2" class="form-control">
              <option value="0">开启</option>
		      <option value="1">关闭</option>
		      <option value="2">反转</option>
		      <option value="3">无动作</option>
		    </select>
      	</div>
	    </template>
	    <!-- 区分 -->
	    <template v-else>
      	<div class="form-group">
              		<label>节点类型</label>
                    <select v-model="button.withDistinction.r0" class="form-control">
                      <option value="0">常开</option>
        		      <option value="1">常闭</option>
        		    </select>
       </div>
      	<div class="form-group">
      		<label>短按反应</label>
            <select v-model="button.withDistinction.r1" class="form-control">
              <option value="0">开启</option>
		      <option value="1">关闭</option>
		      <option value="2">反转</option>
		      <option value="3">无动作</option>
		    </select>
      	</div>
      	<div class="form-group">
      		<label>长按反应</label>
            <select v-model="button.withDistinction.r2" class="form-control">
              <option value="0">开启</option>
		      <option value="1">关闭</option>
		      <option value="2">反转</option>
		      <option value="3">无动作</option>
		    </select>
      	</div>
  			<div class="form-group">
  				<label>长按反应时间2-255(单位100ms)</label>
  				<input  v-model="button.withDistinction.r4" type="text" class="form-control">
  			</div>
      	<div class="form-group">
      		<label>长短按键操作对象</label>
            <select v-model="button.withDistinction.r5" class="form-control">
              <option value="0">1个</option>
		      <option value="1">2个</option>
		    </select>
      	</div>
	    </template>
	  </template>

	  <!-- 开关/调光 -->
	  <template v-else-if="type == '2'">
	    <div class="form-group">
	      <label>节点类型</label>
	      <select v-model="dimmer.r0" class="form-control">
	        <option value="0">常开</option>
	        <option value="1">常闭</option>
	      </select>
	    </div>
	    <div class="form-group">
	      <label>调光功能</label>
	      <select v-model="dimmer.r1" class="form-control">
	        <option value="0">调光加开关</option>
	        <!-- <option value="1">只调光</option>-->
	      </select>
	    </div>
    	<div class="form-group">
    		<label>短按反应</label>
          <select v-model="dimmer.r2" class="form-control">
            <option value="0">开启</option>
	      <option value="1">关闭</option>
	      <option value="2">反转</option>
	      <option value="3">无动作</option>
	    </select>
    	</div>
    	<div class="form-group">
    		<label>长按反应</label>
          <select v-model="dimmer.r3" class="form-control">
            <option value="0">亮度增加</option>
	      <option value="1">亮度减小</option>
	      <option value="2">亮度循环</option>
	    </select>
    	</div>
  		<div class="form-group">
  			<label>长按反应时间3-255(单位100ms)</label>
  			<input  v-model="dimmer.r5" type="text" class="form-control">
  		</div>
  	</template>

	  <!-- 窗帘 -->
	  <template v-else-if="type == '3'">
	    <div class="form-group">
	      <label>节点类型</label>
	      <select v-model="curtain.r1" class="form-control">
	        <option value="0">常开</option>
	        <option value="1">常闭</option>
	      </select>
	    </div>
    	<div class="form-group">
    		<label>长按反应</label>
          <select v-model="curtain.r3" class="form-control">
          <option value="0">上升</option>
	      <option value="1">下降</option>
	      <option value="2">反转</option>
	    </select>
    	</div>
  		<div class="form-group">
  			<label>长按反应时间3-255(单位100ms)</label>
  			<input  v-model="curtain.r4" type="text" class="form-control">
  		</div>
  	</template>

    <footer class="form-footer">
      <label>修改后请点击保存</label>
      <button type="button" class="form-control btn btn-form btn-primary" @click="save">保存</button>
    </footer>
  </form>
</template>

<script>
  import Device from '@/services/device'
  import Channel from '@/services/channeldb'
  import Ini from '@/services/ini'
  export default {
    props: ['name'],
    data () {
      //父组件一次将所有的子组件初始化完毕，不是进入再初始化的
      let curref=this.name
      let inifile = './USU.ini'
      let fileconfig  = Ini.loadFileSync(inifile)
      if(curref=="channelA"){
                 let initdata1=Channel.achannel
                 let curdata = fileconfig[curref];
                 initdata1.firstval = curdata
                 curdata = JSON.parse(curdata)
                 if(curdata){
                  let selectkind = curdata.type
                  initdata1.type = selectkind
                  if(selectkind == 1){
                          initdata1.button.distinction = curdata.distinction
                           if (curdata.distinction === '0') {
                              initdata1.button.withoutDistinction.r1 = curdata.nor1
                              initdata1.button.withoutDistinction.r2 = curdata.nor2
                           }else{
                               initdata1.button.withDistinction.r0 = curdata.yesr0
                               initdata1.button.withDistinction.r1 = curdata.yesr1
                               initdata1.button.withDistinction.r2 = curdata.yesr2
                               initdata1.button.withDistinction.r4 = curdata.yesr4
                               initdata1.button.withDistinction.r5 = curdata.yesr5
                           }
                    }
                    else if(selectkind == 2){
                              initdata1.dimmer.r0 = curdata.dimmer0
                              initdata1.dimmer.r1 = curdata.dimmer1
                              initdata1.dimmer.r2 = curdata.dimmer2
                              initdata1.dimmer.r3 = curdata.dimmer3
                              initdata1.dimmer.r5 = curdata.dimmer5
                    }
                    else if(selectkind == 3){
                              initdata1.curtain.r1 = curdata.curtainr1
                              initdata1.curtain.r3 = curdata.curtainr3
                              initdata1.curtain.r4 = curdata.curtainr4
                    }
                }
                return initdata1
            }
            else if(curref=="channelB"){
                let initdata2=Channel.bchannel
                let curdata = fileconfig[curref];
                initdata2.firstval = curdata
                curdata = JSON.parse(curdata)
                console.log('B',curdata)
                if(curdata){
                let selectkind = curdata.type
                initdata2.type = selectkind
                if(selectkind == 1){
                        initdata2.button.distinction = curdata.distinction
                         if (curdata.distinction === '0') {
                            initdata2.button.withoutDistinction.r1 = curdata.nor1
                            initdata2.button.withoutDistinction.r2 = curdata.nor2
                         }else{
                             initdata2.button.withDistinction.r0 = curdata.yesr0
                             initdata2.button.withDistinction.r1 = curdata.yesr1
                             initdata2.button.withDistinction.r2 = curdata.yesr2
                             initdata2.button.withDistinction.r4 = curdata.yesr4
                             initdata2.button.withDistinction.r5 = curdata.yesr5
                         }
                  }
                  else if(selectkind == 2){
                            initdata2.dimmer.r0 = curdata.dimmer0
                            initdata2.dimmer.r1 = curdata.dimmer1
                            initdata2.dimmer.r2 = curdata.dimmer2
                            initdata2.dimmer.r3 = curdata.dimmer3
                            initdata2.dimmer.r5 = curdata.dimmer5
                  }
                  else if(selectkind == 3){
                            initdata2.curtain.r1 = curdata.curtainr1
                            initdata2.curtain.r3 = curdata.curtainr3
                            initdata2.curtain.r4 = curdata.curtainr4
                  }
                }
                  return initdata2
          }
           else if(curref=="channelC"){
                       let initdata3=Channel.cchannel
                       let curdata = fileconfig[curref];
                       initdata3.firstval = curdata
                       curdata = JSON.parse(curdata)
                       console.log('C',curdata)
                       if(curdata){
                        let selectkind = curdata.type
                        initdata3.type = selectkind
                        if(selectkind == 1){
                                initdata3.button.distinction = curdata.distinction
                                 if (curdata.distinction === '0') {
                                    initdata3.button.withoutDistinction.r1 = curdata.nor1
                                    initdata3.button.withoutDistinction.r2 = curdata.nor2
                                 }else{
                                     initdata3.button.withDistinction.r0 = curdata.yesr0
                                     initdata3.button.withDistinction.r1 = curdata.yesr1
                                     initdata3.button.withDistinction.r2 = curdata.yesr2
                                     initdata3.button.withDistinction.r4 = curdata.yesr4
                                     initdata3.button.withDistinction.r5 = curdata.yesr5
                                 }
                          }
                          else if(selectkind == 2){
                                    initdata3.dimmer.r0 = curdata.dimmer0
                                    initdata3.dimmer.r1 = curdata.dimmer1
                                    initdata3.dimmer.r2 = curdata.dimmer2
                                    initdata3.dimmer.r3 = curdata.dimmer3
                                    initdata3.dimmer.r5 = curdata.dimmer5
                          }
                          else if(selectkind == 3){
                                    initdata3.curtain.r1 = curdata.curtainr1
                                    initdata3.curtain.r3 = curdata.curtainr3
                                    initdata3.curtain.r4 = curdata.curtainr4
                          }
                     }
                      return initdata3
                  }
          else if(curref=="channelD"){
                let initdata4=Channel.dchannel
                let curdata = fileconfig[curref];
                initdata4.firstval = curdata
                curdata = JSON.parse(curdata)
                  console.log('D',curdata)
                  if(curdata){
                  let selectkind = curdata.type
                  initdata4.type = selectkind
                  if(selectkind == 1){
                          initdata4.button.distinction = curdata.distinction
                           if (curdata.distinction === '0') {
                              initdata4.button.withoutDistinction.r1 = curdata.nor1
                              initdata4.button.withoutDistinction.r2 = curdata.nor2
                           }else{
                               initdata4.button.withDistinction.r0 = curdata.yesr0
                               initdata4.button.withDistinction.r1 = curdata.yesr1
                               initdata4.button.withDistinction.r2 = curdata.yesr2
                               initdata4.button.withDistinction.r4 = curdata.yesr4
                               initdata4.button.withDistinction.r5 = curdata.yesr5
                           }
                    }
                    else if(selectkind == 2){
                              initdata4.dimmer.r0 = curdata.dimmer0
                              initdata4.dimmer.r1 = curdata.dimmer1
                              initdata4.dimmer.r2 = curdata.dimmer2
                              initdata4.dimmer.r3 = curdata.dimmer3
                              initdata4.dimmer.r5 = curdata.dimmer5
                    }
                    else if(selectkind == 3){
                              initdata4.curtain.r1 = curdata.curtainr1
                              initdata4.curtain.r3 = curdata.curtainr3
                              initdata4.curtain.r4 = curdata.curtainr4
                    }
              }
                return initdata4
            }
    },
    methods: {
      save () {
        let refname=this.name
        let that=this
        let channelIndex = ['channelA', 'channelB', 'channelC', 'channelD'].indexOf(refname)
        //console.log('channelIndex', channelIndex)
        let storage = window.localStorage
        let inifile = './USU.ini'
        let fs = require('fs');
        let fileconfig = Ini.loadFileSync(inifile)

        let ds = []
        let os = []
        let formdata={}
        formdata.type = this.type
        if (this.type === '0') {
          ds = [0, 0, 0, 0]
        } else if (this.type === '1') {
          //不管怎样是要保持的
          formdata.distinction=this.button.distinction
          if (this.button.distinction === '0') {
            // 开关--不区分
            formdata.nor1=this.button.withoutDistinction.r1
            formdata.nor2=this.button.withoutDistinction.r2
            ds = [0].concat(Device.merge(this.button.withoutDistinction, 7))
            // 只有一个通讯对象
            os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '开关',10))
          } else {
            // 开关--区分
            formdata.yesr0=this.button.withDistinction.r0
            formdata.yesr1=this.button.withDistinction.r1
            formdata.yesr2=this.button.withDistinction.r2
            formdata.yesr4=this.button.withDistinction.r4
            formdata.yesr5=this.button.withDistinction.r5
            ds = [1].concat(Device.merge(this.button.withDistinction, 8))
            if (this.button.withDistinction.r5 === '0') {
              // 只有一个通讯对象
              os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '开关',110))
            } else {
              // 两个通讯对象
              os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '开关[短按]',1111))
              os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '开关[长按]',1112))
            }
          }
        } else if (this.type === '2') {
          // 开关/调光
          formdata.dimmer0=this.dimmer.r0
          formdata.dimmer1=this.dimmer.r1
          formdata.dimmer2=this.dimmer.r2
          formdata.dimmer3=this.dimmer.r3
          formdata.dimmer5=this.dimmer.r5
          ds = Device.merge(this.dimmer, 10)
          if (this.dimmer.r1 === '0') {
            // 调光+开关 两个通讯对象
            os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '开关/调光[开关]',211))
            os.push(new Device.CObject(Device.CObject.CT, Device.CObject.UINT4, channelIndex, '开关/调光[调光]',212))
          } else {
            // 只有调光
            os.push(new Device.CObject(Device.CObject.CT, Device.CObject.UINT4, channelIndex, '开关/调光[只调光]',20))
          }
        } else if (this.type === '3') {
          // 窗帘
          formdata.curtainr1=this.curtain.r1
          formdata.curtainr3=this.curtain.r3
          formdata.curtainr4=this.curtain.r4
          ds = Device.merge(this.curtain, 7)
          // 两个通讯对象
          os.push(new Device.CObject(Device.CObject.CWT, Device.CObject.UINT1, channelIndex, '窗帘[上移/下降]',30))
          os.push(new Device.CObject(Device.CObject.CT, Device.CObject.UINT1, channelIndex, '窗帘[停止]',31))
        }
        //formdata.belongTo = channelIndex
        let steptext=JSON.stringify(formdata);
        fileconfig[refname] = steptext
        storage.setItem(refname,steptext)
        fs.writeFileSync(inifile, fileconfig)

        // 合并
        ds = [parseInt(this.type)].concat(ds)
        //ds协议编码的各个数字
        //console.log(ds)
        this.$emit('submitFrameConfig', new Device.FrameConfig(this.name, ds, os))
      }
    }
  }
</script>