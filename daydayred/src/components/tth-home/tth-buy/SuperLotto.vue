<template>
  <!--大乐透-->
<div class="superLotto_wrap">
  <!--main-->
  <div class="superLotto_main" v-show="isMainShow">
    <!--头-->
    <BuyHeader :runLotto="'/biglt'" :MethodsArr="MethodsArr" :MoreArr="MoreArr" @changeSelectBall="changeSelectBall" @instructionShow="isInstructionShow"></BuyHeader>
    <!--选球-->
    <div class="superLotto_content">
      <!--截止日期-->
      <div class="superLotto_time">
        <strong class="superLotto_time_left">{{MoreTimeArr.Name}} {{MoreTimeArr.BuyEndTime}}</strong>
        <span class="superLotto_time_right" @click="isShowRecent = true">近期开奖</span>
      </div>
      <!--普通投注-->
      <div class="superLotto_normal" v-show="changeBall == 0">
        <!--规则5/2-->
        <div class="superLotto_normal_rule">
          <span>至少选择<b>5</b>个红球 <b class="blueB">2</b>个蓝球</span>
          <div @click="randomNum"><i class="iconfont icon-yaoyiyao"></i>机选</div>
        </div>
        <SelectMoreBall :randomTimes="randomTimes" :randomNums="randomNums" :clearAll="clearAll" @returnNum="getNumRedPT" :maxNum="35" :isRedColor="true"></SelectMoreBall>
        <p class="line"></p>
        <SelectMoreBall :randomTimes="randomTimes" :randomNums="randomNums" :clearAll="clearAll" @returnNum="getNumBluePT" :maxNum="12" :isRedColor="false"></SelectMoreBall>
      </div>
      <!--胆拖投注-->
      <div class="superLotto_dantuo" v-show="changeBall == 1">
        <div class="superLotto_dantuo_rule">
          <span>胆码区-红球,我认为必出的号码,选<b>1-4</b>个</span>
        </div>
        <SelectMoreBall :maxNum="35" :isRedColor="true"></SelectMoreBall>
        <p class="line"></p>
        <div class="superLotto_dantuo_rule">
          <span>拖码区-红球,我认为必出的号码,至少<b>2</b>个</span>
        </div>
        <SelectMoreBall :maxNum="35" :isRedColor="true"></SelectMoreBall>
        <p class="line"></p>
        <div class="superLotto_dantuo_rule">
          <span>胆码区篮球,至多选择<b class="dantuoBlue">1</b>个</span>
        </div>
        <SelectMoreBall :maxNum="12" :isRedColor="false"></SelectMoreBall>
        <p class="line"></p>
        <div class="superLotto_dantuo_rule">
          <span>拖码区-篮球,至少选择<b class="dantuoBlue">2</b>个</span>
        </div>
        <SelectMoreBall :maxNum="12" :isRedColor="false"></SelectMoreBall>
      </div>
    </div>
    <!--尾-->
    <BuyFooter @clearAllNum="clearAllNum" :countZhu="countZhu" :countMoney="countMoney"></BuyFooter>
    <!--隐藏：最近开奖-->
    <mt-popup v-model="isShowRecent" position="bottom" class="recentAward">
      <!--分割符号-->
      <BuyRecentAward :recentArr="recentArr" :isHaveBlue="true"></BuyRecentAward>
    </mt-popup>
  </div>
  <!--隐藏：玩法说明-->
  <BuyInstruction v-show="isInsShow" :thisPage="39" :thisTitle="'大乐透玩法说明'" @instructionClose="isInstructionShow"></BuyInstruction>
</div>
</template>

<script>
  import BuyFooter from '../tth-buy/buy-footer'
  import BuyHeader from '../tth-buy/buy-header'
  import BuyRecentAward from '../tth-buy/buy-recentAward.vue'
  import BuyInstruction from '../tth-buy/buy-instruction.vue'
  import SelectMoreBall from '../tth-buy/buy-selectMoreBall.vue'
  import {Indicator} from 'mint-ui'
  export default {
    name: 'SuperLotto',
    components: {
      BuyFooter,
      BuyHeader,
      BuyRecentAward,
      BuyInstruction,
      SelectMoreBall
    },
    data () {
      return {
        changeBall: 0,
        isMainShow: true,
        isInsShow: false,
        recentArr: [],
        isShowRecent: false,
        MoreTimeArr: [],
        MethodsArr: ['大乐透普通投注', '大乐透胆拖投注'],
        MoreArr: [
          {
            'moreName': '近期开奖',
            'moreIndex': 'kaijiang'
          },
          {
            'moreName': '显示遗漏',
            'moreIndex': 'yilou'
          },
          {
            'moreName': '走势图表',
            'moreIndex': 'zoushi'
          },
          {
            'moreName': '玩法说明',
            'moreIndex': 'shuoming'
          }
        ],
        numArrPT: [[], []],
        countZhu: 0,
        countMoney: 0,
        clearAll: false,
        buyCount: [0, 0, 0, 0, 1, 7, 28, 84, 210, 462, 924, 1716, 3003, 5005, 80081, 12376, 18564, 27132, 38760, 54264, 74613, 100947, 134596, 177100, 230230, 296010, 376739.999, 475019.999, 593774.999, 736281, 906192, 1107568],
        randomNums: false,
        randomTimes: 0
      }
    },
    methods: {
      fecthSuperLottoData () {
        this.$request({
          type: 'get',
          url: '/api/Data/Handler.ashx?action=201&params={%22LotteryID%22:%224%22}',
          success: function (res) {
            this.MoreTimeArr = res.data.data
//            console.log(res.data.data)
            Indicator.close()
          },
          failed: function (err) {
            console.log('未找到大乐透数据:' + err)
          }
        })
      },
      fecthSuperLottoRecentData () {
        this.$request({
          type: 'get',
          url: '/api/Data/Handler.ashx?action=602&params={%22LotteryID%22:%224%22}',
          success: function (res) {
            this.recentArr = res.data.data
//            console.log(res.data.data)
          },
          failed: function (err) {
            console.log('未找到排列3最近开奖数据:' + err)
          }
        })
      },
      changeSelectBall (index) {
        this.changeBall = index
        console.log(index)
      },
      // 玩法介绍显示
      isInstructionShow () {
        this.isMainShow = !this.isMainShow
        this.isInsShow = !this.isInsShow
      },
      // 计算总金额和总注数
      getPTCount () {
        let num1 = this.numArrPT[0].length
//        console.log('红球个数' + this.numArrPT[0].length)
        let num2 = this.numArrPT[1].length
        num1 = this.buyCount[num1 - 1]
        num2 = (num2 - 1) * num2 / 2
        this.countZhu = num1 * num2
        this.countMoney = this.countZhu * 2
//        console.log(this.countMoney)
      },
      // 获取选球组件返回的数组
      getNumRedPT (num) {
        this.numArrPT[0] = num
        this.getPTCount()
      },
      getNumBluePT (num) {
        this.numArrPT[1] = num
        this.getPTCount()
      },
      // 清除所有选择的球
      clearAllNum () {
        this.clearAll = !this.clearAll
        this.countZhu = 0
        this.countMoney = 0
        console.log('清除')
      },
      // 随机球
      randomNum () {
        this.randomNums = !this.randomNums
        this.randomTimes = 5
      }
    },
    mounted () {
      Indicator.open('加载中...')
      this.fecthSuperLottoData()
      this.fecthSuperLottoRecentData()
    }
  }
</script>

<style scoped lang="less">
    @import "../../../common/css/style.less";
    .superLotto_wrap{
      width: 100%;
      height: 667px;
      position: relative;
      background-color: @color-background-gray;
      .superLotto_content{
        height: 100%;
        width: 100%;
        background-color: #FFFFFF;
        padding-bottom: 16.33333vmin;
        /*时间*/
        .superLotto_time {
          background-color: @color-background-gray;
          display: flex;
          justify-content: space-between;
          align-items: center;
          width: 100%;
          height: 8.8vmin;
          box-sizing: border-box;
          padding: 0 2.66667vmin;
          margin-top: 12vmin;
          font-size: 3.2vmin;
          .superLotto_time_left {
            color: @color-text-black;
            font-weight: bolder;
          }
          .superLotto_time_right {
            color: #6b8dff;
          }
        }
        /*普通*/
        .superLotto_normal{
          width: 100%;
          .superLotto_normal_rule{
            box-sizing: border-box;
            font-size: 3.2vmin;
            width: 100%;
            height: 8vmin;
            padding: 0 3.2vmin;
            display: flex;
            align-items: center;
            justify-content: space-between;
            background-color: white;
            color: @color-text-gray;
            span{
              b{
                color: @color-red;
              }
              .blueB{
                color: @color-blue;
              }
            }
            div{
              display: flex;
              align-items: center;
              justify-content: center;
              i{
                margin-right: 1vmin;
                font-size: 7vmin;
                transform: rotate(45deg);
              }
            }
          }
        }
        /*胆拖*/
        .superLotto_dantuo{
          width: 100%;
          .superLotto_dantuo_rule{
            font-size: 3.2vmin;
            width: 100%;
            height: 8vmin;
            padding: 0 3.2vmin;
            display: flex;
            justify-content: flex-start;
            align-items: center;
            span{
              color: @color-text-gray;
              b{
                color: @color-red;
              }
              .dantuoBlue{
                color: @color-blue
              }
            }
          }
        }
        /*分割线*/
        .line{
          width: 100%;
          height: .66667vmin;
          /*display: block;*/
          background: #e6e6e6;
          border-top: 2.66667vmin solid white;
          border-bottom: 2.66667vmin solid white;
        }
      }
    }
</style>
