<script>
import Axios from '@utils/axios'
import moment from 'moment'
import ButtonLoading from '@components/utils/button-loading'
import { BroadcastChannel } from 'broadcast-channel'

export default {
  components: { 
    ButtonLoading
  },
  props: {
    displayTitle: {
      type: Boolean,
      required: false,
      default: false,
    },
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
    scope: {
      type: String,
      required: false,
      default: 'board',
    },
  },
  data() {
    return {
      loading: false,
      time: '00:00:00',
      btnStart: true,
      btnStop: false,
      seconds: 0,
      clockIsReady: false,
      timestampStarted: 0,
      key: 'task-' + this.task.uuid,
      started: false,
      clock: false,
      startedUTC: false,
      channel: null,
      modalClockTime: '00:00:00',
      timerLogComment: '',
      timeTrackerPreloaded: false,
    }
  },
  watch: {
    close: function(val) {
      this.reset()
    },
  },
  mounted() {
    this.getTimers(this.task)
  },
  beforeDestroy() {},
  created() {
    this.channel = new BroadcastChannel(this.key)
    this.channel.onmessage = (event) => {
      // CONVERT RAWUTC TO MOMENT
      event.startedUTC = moment.utc(event.startedRawUTC)
      this.handleTaskTimerEvent(event)
    }
    
    if (this.task.time_tracker !== null) {
      this.clockStartFrom(moment.utc(this.task.time_tracker))
      this.timeTrackerPreloaded = true
      this.clockIsReady = true
    } else {
      this.timeTrackerPreloaded = true
      this.clockIsReady = true
    }
  },
  methods: {
    clockStart(seconds) {
      if (this.started === false) {
        this.btnStart = false
        this.btnStop = true
        this.seconds = seconds
        this.started = true
        this.clock = setInterval(this.clockTicker, 1000)
        this.startedUTC = moment.utc(
          moment()
            .utc()
            .format('YYYY-MM-DD HH:mm:ss')
        )
      }
    },
    clockStop() {
      if (this.started === true) {
        this.btnStart = true
        this.btnStop = false
        this.time = '00:00:00'
        this.started = false
        this.task.time_tracker = null
        clearInterval(this.clock)
      }
    },
    clockStartFrom(startedUTC) {
      let UTCNow = moment.utc(
        moment()
          .utc()
          .format('YYYY-MM-DD HH:mm:ss')
      )
      var duration = moment.duration(UTCNow.diff(startedUTC))
      this.seconds = duration.asSeconds()

      this.startedUTC = startedUTC
      this.btnStart = false
      this.btnStop = true
      this.started = true

      this.clock = setInterval(this.clockTicker, 1000)
    },
    handleTaskTimerEvent(event) {
      let status = event.status
      if (this.task.uuid === event.uuid) {
        if (status === 'start' && this.started === false) this.clockStartFrom(event.startedUTC)
        else if (status === 'end' && this.started === true) this.clockStop()
        else if (status === 'cancel' && this.started === true) this.clockStop()
      }
    },
    init() {
      this.reset()
    },
    clickhandler() {},
    getTimers(task) {
      if (!this.timeTrackerPreloaded) {
        Axios()
          .get(
            '/time-trackings/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug +
              '&task_uuid=' +
              this.task.uuid
          )
          .then((response) => {
            this.clockIsReady = true
            if (
              response.data.resume !== undefined &&
              response.data.resume.period_start !== null &&
              response.data.resume.period_end === null
            ) {
              this.clockStartFrom(moment.utc(response.data.resume.period_start))
            }
          })
          .catch((e) => {
            console.error(e)
          })
      }
    },
    sendTime(status) {
      Axios()
        .post(
          'timer/' + status + '/?company_slug=' + this.task.company.slug + '&project_slug=' + this.task.project.slug,
          {
            task_uuid: this.task.uuid,
            comment: this.timerLogComment,
          }
        )
        .then((response) => {
          this.channel.postMessage({
            uuid: this.task.uuid,
            status: status,
            key: this.key,
            startedRawUTC: moment.utc(this.startedUTC).format('YYYY-MM-DD HH:mm:ss'),
          })
          this.task.time_tracker = moment.utc(this.startedUTC).format('YYYY-MM-DD HH:mm:ss')
        })
        .catch((e) => {
          console.error(e)
        })
    },
    start() {
      this.clockStart(0)
      this.sendTime('start')
    },
    stop() {
      this.$refs['modal'].show()
      this.modalClockTime = this.time
      this.task.time_tracker = null
    },
    confirmTimeSave() {
      this.clockStop()
      this.sendTime('end')
      this.timerLogComment = ''
      this.task.time_tracker = null
      this.task.stats.time_trackers = this.task.stats.time_trackers + 1
      this.closeModal(this.$refs['modal'])
    },
    cancelTimeSave() {
      this.clockStop()
      this.sendTime('cancel')
      this.timerLogComment = ''
      this.task.time_tracker = null
      this.closeModal(this.$refs['modal'])
    },
    reset() {
      this.clockStop()
      clearInterval(this.clockTicker)
    },
    clockTicker() {
      this.seconds += 1

      let UTCNow = moment.utc(
        moment()
          .utc()
          .format('YYYY-MM-DD HH:mm:ss')
      )
      this.time = moment.utc(UTCNow.diff(this.startedUTC)).format('HH:mm:ss')
    },
  },
}
</script>

<template>
  <div class="badge badge-light mb-1 task-timer timer-content" @click.stop.prevent="clickhandler">
    <h5 v-if="displayTitle" class="mg-b-10">{{ $t('Task Time Tracking') }}</h5>
    <b-spinner v-if="!clockIsReady" v-show="!clockIsReady" :label="$t('Loading')" variant="secondary" small></b-spinner>
    <div v-else class="d-flex justify-content-between ">
      <div id="time">{{ time }}</div>
      <div class="ml-1">
        <font-awesome-icon 
        v-show="btnStart && clockIsReady" 
        class="cursor-pointer" 
        :icon="['fas', 'play']"
        @click.stop.prevent="start" />
        <font-awesome-icon 
        v-show="btnStop && clockIsReady" 
        class="cursor-pointer" 
        :icon="['fas', 'stop']"
        @click.stop.prevent="stop" />
       </div>
    </div>
    <b-modal
      id="modal-black"
      ref="modal"
      class="modalConfirmBody"
      size="xxx" hide-header hide-footer>
      <b-container>
        <b-row class="mb-3">
          <b-col>
            <h3 class="mb-0">{{ $t('Stop work log time on task') }}</h3>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <h5>{{ task.title }}</h5>
            <hr>
            <h6><strong>{{ $t('Work Log') }}</strong>: {{ modalClockTime }}</h6>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-form-textarea
            v-model="timerLogComment"
            :placeholder="$t('Describe how time was used for this task')"
            trim
            rows="3"
            max-rows="6"></b-form-textarea>
          </b-col>
        </b-row>
        <b-row class="mt-2">
          <b-col align-self="center">
            <b-link 
              v-show="!loading" 
              @click="cancelTimeSave" 
              v-text="$t('Discard this time spent')" />
          </b-col>
          <b-col class="text-right">
            <ButtonLoading
              :loading="loading"
              :title="$t('Confirm')"
              :title-loading="$t('Adding')"
              type="btn-md"
              mode="button"
              @action="confirmTimeSave"></ButtonLoading>
          </b-col>
        </b-row>
      </b-container>
    </b-modal>
  </div>
</template>
