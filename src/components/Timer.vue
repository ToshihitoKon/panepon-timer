<template>
  <div>
    <h1>{{stateToString()}}</h1>
    <h3>{{displayTime}}</h3>
    <table>
      <tr>
        <th>stage</th>
        <th>rap time</th>
        <th>total time</th>
      </tr>
      <tr v-for="time,index in times"
        :key="index"
      >
        <td>
          stage{{time.stageCount}}
        </td>
        <td>
          {{toDisplayTime(time.time)}}
        </td>
        <td>
          {{toDisplayTime(time.checkpointTime)}}
        </td>
      </tr>
    </table>
    <div class="input-group mt-4">
      <textarea v-model="resultText" id="result" class="form-control ignore-press-timer" style="white-space: pre" />
      <div class="input-group-prepend">
        <button type="button" v-on:click="copoyResult" class="btn btn-info ignore-press-timer">結果をｺﾎﾟｲｰ</button>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data: function() {
    return {
      times: [],
      currentTimerProps: {
        startAt: 0,
        stopAt: 0,
      },
      keyDownState: 0,
      keyUpState: 0,
      now: 0,
      resultText: '',
      stageCount: 1,
    }
  },
  computed: {
    currentState: function() {
      return this.keyDownState + this.keyUpState
    },
    displayTime: function() {
      switch(this.currentState){
      case 0:
      case 1:
        return 'ready'
      case 25:
        return this.toDisplayTime(this.times[this.times.length - 1].checkpointTime)
      default:
        return this.toDisplayTime(this.now - this.currentTimerProps.startAt)
      }
    },
  },
  mounted () {
    this.setTitle(),
    this.watchNowLoop(),
    this.setKeyboardHandler(),
    this.setTouchHandler()
  },
  methods:{
    // mount時設定
    setTitle: function() {
      document.title = "パネポン"
    },
    watchNowLoop: function() {
      window.requestAnimationFrame(this.watchNowLoop)
      this.watchNow()
    },
    watchNow: function () {
      this.now = Date.now()
    },
    setKeyboardHandler: function() {
      window.addEventListener('keydown', this.pressTimer)
      window.addEventListener('keyup', this.pressTimer)
    },
    setTouchHandler: function() {
      window.addEventListener('touchstart',this.pressTimer, false)
      window.addEventListener('touchend',this.pressTimer, false)
    },

    // 本操作
    pressTimer: function (e) {
      // ignore-press-timerクラスが付いてるものをタップした時は無視
      if (e.target.classList.contains('ignore-press-timer')) return

      // stateが25（time stop）後は無視
      if (25 <= this.currentState) return

      switch (e.type) {
        case 'keydown':
          if ( e.code != 'Space' || e.repeat ) return
          this.downTimer()
          break
        case 'keyup':
          this.upTimer()
          break
        case 'touchstart':
          if ( e.touches.length != 1 ) return
          this.downTimer()
          break
        case 'touchend':
          if ( e.touches.length != 0 ) return
          this.upTimer()
          break
        default:
          return
      }
    },
    downTimer: function () {
      // up/downが対応しなかった場合の不整合対策
      if (this.keyUpState == this.keyDownState) {
        this.keyDownState += 1
      }

      this.actionTimer(this.keyUpState + this.keyDownState)
    },
    upTimer: function () {
      this.keyUpState = this.keyDownState

      this.actionTimer(this.keyUpState + this.keyDownState)
    },
    actionTimer: function(state) {
      switch (state) {
      case 2: // Key up, Start
        this.startTimer()
        break;
      case 3:  // stage1 clear
      case 5:  // stage2 clear
      case 7:  // stage3 clear
      case 9:  // stage4 clear
      case 11: // stage5 clear
      case 13: // stage6 clear
      case 15: // stage7 clear
      case 17: // stage8 clear
      case 19: // stage9 clear
      case 21: // stage10 clear
      case 23: // stage11 clear
        this.rapTimer()
        break;
      case 25: // stage12 clear timer stop
        this.stopTimer()
        break;
      }
    },

    // actions
    startTimer: function () {
      const timer = this.currentTimerProps
      timer.startAt = Date.now()
      this.currentTimerProps = timer
    },
    rapTimer: function () {
      const times = this.times
      const stageCount = this.stageCount

      const checkpointTime = Date.now() - this.currentTimerProps.startAt
      var beforeCheckpointTime = 0
      if (times.length != 0) {
        beforeCheckpointTime = times[times.length - 1].checkpointTime
      }

      times.push({
        time: checkpointTime - beforeCheckpointTime,
        stageCount: stageCount,
        checkpointTime: checkpointTime,
      })
      this.times = times
      this.stageCount = stageCount + 1
    },
    stopTimer: function () {
      const timer = this.currentTimerProps
      timer.stopAt = Date.now()
      this.currentTimerProps = timer
      this.rapTimer()

     this.setResultText()
    },

    // filtersみたいなやつら
    stateToString: function () {
      switch(this.currentState) {
      case 0:
      case 1:
        return 'ready'
      case 2:
        return 'stage 1'
      case 3:
      case 4:
        return 'stage 2'
      case 5:
      case 6:
        return 'stage 3'
      case 7:
      case 8:
        return 'stage 4'
      case 9:
      case 10:
        return 'stage 5'
      case 11:
      case 12:
        return 'stage 6'
      case 13:
      case 14:
        return 'stage 7'
      case 15:
      case 16:
        return 'stage 8'
      case 17:
      case 18:
        return 'stage 9'
      case 19:
      case 20:
        return 'stage 10'
      case 21:
      case 22:
        return 'stage 11'
      case 23:
      case 24: 
        return 'stage 12'
      case 25:
        return 'おつかれ'
      default:
       return 'unknown'
      }
    },
    toDisplayTime: function(ms) {
      const min = Math.floor(ms / (1000 * 60)) % 60
      const sec = Math.floor(ms / 1000) % 60
      const msec = ms % 1000
      return this.zeroPad(min,2) + ':' + this.zeroPad(sec,2) + '.' + this.zeroPadEnd(Math.floor(msec / 10), 2)
    },

    // utils
    zeroPad: function(num, len){
      return (''+num).padStart(len,'0')
    },
    zeroPadEnd: function(num, len){
      return (''+num).padEnd(len,'0')
    },
    setResultText: function() {
      var text = ''
      this.times.map((t)=>{
        text += 'stage' +  (t.stageCount) + ': ' + this.timeToResultText(t) + '\n'
      })
      text += '\n' + 'time: ' +  this.toDisplayTime(this.times[this.times.length-1].checkpointTime) 
      this.resultText = text
    },
    timeToResultText: function(time) {
      return this.toDisplayTime(time.time)
    },
    copoyResult: function() {
      const result = document.getElementById('result')
      result.select()
      document.execCommand('copy')
    }
  }
}

</script>
