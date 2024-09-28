<script>
import moment from "moment";
import 'moment/locale/ja';

export default {
  components: {
  },
  data() {
    return {
      alarms: [],
      worker: null,
    }
  },
  watch: { },
  computed: {
    nextAlarm: function() {
      return this.alarms[0];
    }
  },
  methods: {
    createAlarms(intervalMinutes) {
      let alarms = [];
      let cur = moment().hours(0).minutes(0).seconds(0).millisecond(0);
      let now = moment();
      let interval = intervalMinutes;
      let totalMinutes = 24 * 60;
      for(let i = 0; i < totalMinutes/interval; i++) {
        if(now.diff(cur) < 0) {
          alarms.push({ id: i, time: cur.toDate().toISOString() });
        }
        cur.add(interval,"minutes");
      }
      return alarms;
    },
    stringifyTime(time) {
      if(time == null || time === undefined) return "";
      return moment(new Date(Date.parse(time))).format("YYYY-MM-DD HH:mm:ss");
    }
  },
  mounted() {
    var blob = new Blob([
      document.querySelector('#worker1').textContent
    ], { type: "text/javascript" })

    // Note: window.webkitURL.createObjectURL() in Chrome 10+.
    // var worker = new Worker(window.URL.createObjectURL(blob));
    this.worker = new Worker(window.webkitURL.createObjectURL(blob));
    // worker.onmessage = function(e) {
    //   console.log("Received: " + e.data);
    // }
    // worker.postMessage("hello"); // Start the worker.




    Notification.requestPermission();
    // this.worker = new Worker("worker.js");
    this.worker.addEventListener("message", (e) => {
      console.log("worker");
      this.alarms = JSON.parse(e.data.alarms);
    });

    let alarms = this.createAlarms(15);
    this.worker.postMessage({ alarms: JSON.stringify(alarms)});
    this.alarms = alarms;
  }
}
</script>

<template>
  next: {{ stringifyTime(nextAlarm?.time) }}
</template>

<style scoped>
</style>
