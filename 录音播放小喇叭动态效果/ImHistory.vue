<template>
    <div>
        <!--{{dataForm}}-->
        <div class="history-main" style="overflow:auto;text-align:left;height:400px" id="history-main">
            <ul v-if="historyData.length !=0">
                <template v-for="item in historyData">
                    <li v-if="item.userName.indexOf('|') > -1" :key="item">
                        <template v-if="item.contentType === '0'">
                            <span style='color:#0b8bf0'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;" v-html="item.content"></p>
                        </template>
                         <template v-if="item.contentType === '1'">
                            <span style='color:#0b8bf0'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;"><img :src="dealImgUrl(item.url+item.imgurlsmall)" alt="" @click="showBigimg" :data-imgurl="dealImgUrl(item.url+item.imgurlbig)"></p>
                        </template>
                        <template v-if="item.contentType === '2'">
                            <span style='color:#0b8bf0'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;">发送文件:{{item.content}}</p>
                        </template>
                        <template v-if="item.contentType === '101'">
                            <span style='color:#0b8bf0'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <voiceHis :voiceObj="item" :thisVoice="thisVoice" @voiceMth="voiceMth"></voiceHis>
                        </template>
                    </li>
                    <li v-else :key="item">
                        <template v-if="item.contentType === '0'">
                            <span style='color:#91c592'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;" v-html="item.content"></p>
                        </template>
                        <template v-if="item.contentType === '1'">
                            <span style='color:#91c592'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;"><img :src="dealImgUrl(item.url+item.imgurlsmall)" alt="" @click="showBigimg" :data-imgurl="dealImgUrl(item.url+item.imgurlbig)"></p>
                        </template>
                        <template v-if="item.contentType === '2'">
                            <span style='color:#91c592'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <p style="word-wrap:break-word;word-break:break-all;">发送文件:{{item.content}}</p>
                        </template>
                        <template v-if="item.contentType === '101'">
                            <span style='color:#0b8bf0'><i>{{getcreatime(item.beginTime)}}</i>&nbsp;&nbsp;&nbsp;{{item.userNameDisplay}}说</span>
                            <voiceHis :voiceObj="item" :thisVoice="thisVoice" @voiceMth="voiceMth"></voiceHis>
                        </template>
                    </li>
                </template>
            </ul>
            <p v-else>当前没有查询到相关的聊天记录</p>
        </div>
        <!--<div v-if="historyData.length !=0">-->
        <div>
            <span>开始日期:</span>
            <el-date-picker
                v-model="searchForm.startTimeStr"
                type="date"
                placeholder="选择日期"
                format="yyyy-MM-dd"
                @change="dataChangeCreate"
                >
            </el-date-picker>
            <span>结束日期:</span>
            <el-date-picker
                v-model="searchForm.endTimeStr"
                type="date"
                placeholder="选择日期"
                format="yyyy-MM-dd"
                @change="dataChangeEnd"
                 >
            </el-date-picker>
            <el-button type="primary" @click="refreshHistory">查询</el-button>
        </div>
        <div class="pagination" style="text-align:center;" v-if="historyData.length !=0">
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
            :current-page="searchForm.current" :page-sizes="[20, 30, 40, 50]" :page-size="searchForm.limit"
            layout="total, sizes, prev, pager, next, jumper" :total="searchForm.totalnumb" >
            </el-pagination>
        </div>
         <el-dialog  title="图片展示" :visible.sync="dialogTableVisibleHistory"  size="large" :modal=false >
                <div style ="position: relative;width:100%;">
                    <img :src="imgUrlhistory" alt="" v-if="dialogTableVisibleHistory" style="position:relative;left:50%;transform: translateX(-50%);width:100%;">
                </div>
        </el-dialog>
    </div>
</template>
<script>
import cache from '../../../utils/cache.js'
import voiceHis from './voiceHistory'
let ipccUrl = cache.getItem('ipccaddr')
// eslint-disable-next-line
let hsipccUrl = sysConfig.hsipccUrl
// eslint-disable-next-line
let isHsipccUsed = sysConfig.isHsipccUsed
export default {
  props: ['dataForm'],
  data () {
    return {
      historyData: [],
      searchForm: {
        jobId: this.dataForm.jobId,
        agentServiceId: this.dataForm.agentServiceId,
        channelId: this.dataForm.souceType,
        totalnumb: 0,
        limit: 20,
        current: 1,
        start: 0,
        startTimeStr: '',
        endTimeStr: '',
        startTime: '',
        endTime: '',
        source: ''
      },
      dialogTableVisibleHistory: false,
      imgUrlhistory: '',
      thisVoice: '', // 当前点击的录音
    }
  },
  components: {
    voiceHis
  },
  created () {
    this.searchForm.endTime = this.getNowFormatDate() + ' 23:59:59'
    this.searchForm.startTime = this.getLastYearFormatDate() + ' 00:00:00'
    this.searchForm.endTimeStr = this.getNowFormatDate()
    this.searchForm.startTimeStr = this.getLastYearFormatDate()
    this.refreshHistory()
  },
  methods: {
    dealImgUrl (url) {
      if (domainConfig.urlObj.ipccaddrs.length > 0) {
        domainConfig.urlObj.ipccaddrs.map(addr => {
          if (url.indexOf(addr.oldIp) > -1) {
            return url.replace(addr.oldIp, addr.newIp)
          }
        })
      }
    },
    showBigimg (event) {
      let imgUrl = (event.srcElement || event.target).dataset['imgurl']
      this.imgUrlhistory = imgUrl
      this.dialogTableVisibleHistory = true
    },
    dataChangeCreate (val) { // 开始日期
      this.searchForm.startTime = val + ' 00:00:00'
    },
    dataChangeEnd (val) { // 结束日期
      this.searchForm.endTime = val + ' 23:59:59'
    },
    getNowFormatDate () {
      let date = new Date()
      var seperator1 = '-'
      var year = date.getFullYear()
      var month = date.getMonth() + 1
      var strDate = date.getDate()
      if (month >= 1 && month <= 9) {
        month = '0' + month
      }
      if (strDate >= 0 && strDate <= 9) {
        strDate = '0' + strDate
      }
      var currentdate = year + seperator1 + month + seperator1 + strDate
      return currentdate
    },
    getLastYearFormatDate () { // 当前时间15天前
      var time = new Date()
      var currtime = time.getTime() - 3600 * 1000 * 24 * 15
      var date = new Date(currtime)
      var seperator1 = '-'
      var year = date.getFullYear()
      var month = date.getMonth() + 1
      var strDate = date.getDate()
      if (month >= 1 && month <= 9) {
        month = '0' + month
      }
      if (strDate >= 0 && strDate <= 9) {
        strDate = '0' + strDate
      }
      var currentdate = year + seperator1 + month + seperator1 + strDate
      return currentdate
    },
    voiceMth (val) {
      this.thisVoice = val
    },
    refreshHistory () {
      let self = this
      let jsondata = {}
      if (self.dataForm.messageType === 'imchat') {
        if (isHsipccUsed) { // 部署了渠道协同
          let useId = self.dataForm.weixinNo// 以此判断来源，true是来自微信
          let formatFromNumber = ''
          if (!useId) {
            let useStr = self.dataForm.myfromNumber
            let patter = /_(.+?)-/g
            if (useStr.match(patter) === null) { // 不符合webchat接入时的正则，则表示是H5
              useId = self.dataForm.usernme
            } else {
              useId = useStr.match(patter)[0].slice(1, -1)
            }
            formatFromNumber = self.dataForm.myfromNumber
          }
          jsondata = { // getWebchatHisByPage请求参数
            pageNo: self.searchForm.current,
            pageSize: self.searchForm.limit,
            fromNumber: useId,
            formatFromNumber: formatFromNumber,
            company: localStorage.ipccDeptId,
            startTime: self.searchForm.startTime,
            endTime: self.searchForm.endTime,
            channelId: self.searchForm.channelId
          }
        } else {
          jsondata = { // portal/get_sms_details接口的请求参数
            source: self.searchForm.source,
            fromNumber: self.dataForm.myfromNumber,
            toNumber: self.dataForm.mytoNumber,
            limit: self.searchForm.limit,
            start: self.searchForm.start,
            startTime: self.searchForm.startTime,
            endTime: self.searchForm.endTime
          }
        }
      } else {
        if (undefined === this.searchForm.jobId || undefined === this.searchForm.agentServiceId) {
          return false
        }
        jsondata = {
          source: self.searchForm.source,
          jobId: self.searchForm.jobId,
          agentServiceId: self.searchForm.agentServiceId,
          limit: self.searchForm.limit,
          start: self.searchForm.start,
          startTime: self.searchForm.startTime,
          endTime: self.searchForm.endTime
        }
      }
      if (isHsipccUsed) { // 如果没部署渠道协同，用ipcc的接口查历史记录
        // eslint-disable-next-line\

        $.ajax({
          //url: hsipccUrl + '/getWebchatHisByPage.do', // 新查询接口，请求方式为post
          url: hsipccUrl + '/dialog/getDialogHisByPage',
          // url: 'http://' + localStorage.getItem('ipccaddr') + '/agentserver/services/portal/get_sms_details',  //旧查询接口，请求方式为get
          type: 'post',
           headers: {
                    'Content-Type': "application/json; charset=utf-8"
          },
          data: JSON.stringify(jsondata),
          success: function (data) {
            console.info('+++++++++++++++++++++++++data', data)
            if (data.success) {
              self.searchForm.totalnumb = data.totalCount
              let oneHistoryTalk = data.obj
              for (let i = 0; i < data.obj.length; i++) {
                if (oneHistoryTalk[i].contentType === '0') {
                  self.transferImg(data.obj[i])
                }
                if (oneHistoryTalk[i].contentType === '1') {
                  let arr = oneHistoryTalk[i].content.split('|')
                  // let imgurlbig = arr[0]
                  oneHistoryTalk[i].imgurlbig = arr[0]
                  // let imgurlsmall = arr[1]
                  oneHistoryTalk[i].imgurlsmall = arr[1]
                }
                if (oneHistoryTalk[i].contentType === '101') {
                  oneHistoryTalk[i].voiceInfo = JSON.parse(oneHistoryTalk[i].content)
                }
              }
              let account = localStorage.getItem('ipccDeptId')
              oneHistoryTalk.forEach(function (item, index) {
                item.userNameDisplay = item.userNameDisplay.replace(account, '')
              })
              console.log('======================================++++++++++++++++++++++++++++oneHistoryTalk', oneHistoryTalk)
              self.historyData = oneHistoryTalk
            }
          },
          error: function (xhr, textStatus, errorThrow) {
            console.log(xhr.readyState)
          }
        })
      } else {
        // eslint-disable-next-line
        $.ajax({
          url: 'http://' + ipccUrl + '/agentserver/services/portal/get_sms_details',
          type: 'get',
          headers: {
                    ContentType: "application/json; charset=utf-8"
          },
          data: jsondata,
          success: function (data) {
            console.info('+++++++++++++++++++++++++data', data)
            if (data.resultCode === '0') {
              self.searchForm.totalnumb = data.totalCount
              let oneHistoryTalk = data.topics
              for (let i = 0; i < data.topics.length; i++) {
                if (oneHistoryTalk[i].contentType === '0') {
                  self.transferImg(data.topics[i])
                }
                if (oneHistoryTalk[i].contentType === '1') {
                  let arr = oneHistoryTalk[i].content.split('|')
                  // let imgurlbig = arr[0]
                  oneHistoryTalk[i].imgurlbig = arr[0]
                  // let imgurlsmall = arr[1]
                  oneHistoryTalk[i].imgurlsmall = arr[1]
                }
                if (oneHistoryTalk[i].contentType === '101') {
                  oneHistoryTalk[i].voiceInfo = JSON.parse(oneHistoryTalk[i].content)
                }
              }
              let account = localStorage.getItem('account')
              oneHistoryTalk.forEach(function (item, index) {
                item.userNameDisplay = item.userNameDisplay.replace(account, '')
              })
              console.log('======================================++++++++++++++++++++++++++++oneHistoryTalk', oneHistoryTalk)
              self.historyData = oneHistoryTalk
            }
          },
          error: function (xhr, textStatus, errorThrow) {
            console.log(xhr.readyState)
          }
        })
      }
    },
    handleSizeChange (val) { // 分页标签的每页条数改变事件
      this.searchForm.limit = val
      this.searchForm.start = 0
      this.refreshHistory()
    },
    handleCurrentChange (val) { // 分页标签的页数跳转改变事件
      // console.log('val', val)
      // console.log('val', (val-1) * this.searchForm.limit)
      this.searchForm.current = val
      this.searchForm.start = (val - 1) * this.searchForm.limit
      this.refreshHistory()
    },
    transferImg: function (obj) {
      if (obj.content.match(/[:[0-9]+:]/g)) {
        var arr1 = obj.content.match(/[:[0-9]+:]/g).join(',')
        var arr2 = obj.content.match(/[:[0-9]+:]/g).join(',')
        var num = arr1.indexOf('[')
        var temp = ''
        if (num) {
          arr1 = arr1.substring(num)
          temp = arr2.substring(0, num)
        }
        var arr = arr1.split(',')
        var nuArr = []
        for (var n = 0; n < arr.length; n++) {
          nuArr.push('<img src="static/img/face/' + arr[n].substring(2, arr[n].length - 2) + '.gif"/>')
        }
        var s = obj.content
        for (var j = 0; j < nuArr.length; j++) {
          s = s.replace(/[:[0-9]+:]/, nuArr[j])
        }
        if (temp) {
          s = temp + s
        }
        obj.content = s
      }
    },
    getcreatime: function (creatime) { // 日期表现形式的转化函数
      var date = new Date(creatime)
      var year = date.getFullYear()
      var month = date.getMonth() + 1
      if (month < 10) {
        month = '0' + month
      }
      var day = date.getDate()
      if (day < 10) {
        day = '0' + day
      }
      var hours = date.getHours()
      if (hours < 10) {
        hours = '0' + hours
      }
      var minute = date.getMinutes()
      if (minute < 10) {
        minute = '0' + minute
      }
      var second = date.getSeconds()
      if (second < 10) {
        second = '0' + second
      }
      var str = year + '-' + month + '-' + day + '--' + hours + ':' + minute + ':' + second
      return str
    }
  }
}
</script>
<style src="../sysmanager/SysConfig.css">
</style>
