<script setup>
import { onUnmounted, onMounted, ref } from 'vue';

const props = defineProps(
  {
    searchHistory: {
      type: Object,
      required: true,
    },
  },
  { deep: true }
);

function calcTime(offset) {
  const d = new Date();
  const utc = d.getTime() + d.getTimezoneOffset() * 60000;
  const nd = new Date(utc + offset * 1000);
  const hours = nd.getHours();
  const minutes = nd.getMinutes();
  const seconds = nd.getSeconds();
  return [hours, minutes, seconds];

  // var d = new Date();
  // var utc = d.getTime() + d.getTimezoneOffset() * 60000;
  // var nd = new Date(utc + 3600000 * offset);
  // var hours = nd.getHours();
  // var minutes = nd.getMinutes();
  // var seconds = nd.getSeconds();
  // return [hours, minutes, seconds];
}

const deg = 6;
const timeZoneId = ref();
const timeZoneName = ref();
const timePeriod = ref('');
let intervalId;

onMounted(async () => {
  const apiKey = import.meta.env.VITE_GoogleMAP_API_KEY;
  const timestamp = Math.round(new Date().getTime() / 1000);
  const response = await fetch(
    `https://maps.googleapis.com/maps/api/timezone/json?location=${props.searchHistory.lat},${props.searchHistory.lng}&timestamp=${timestamp}&key=${apiKey}`
  );
  const data = await response.json();
  intervalId = setInterval(async () => {
    const [hours, minutes, seconds] = calcTime(data.dstOffset + data.rawOffset);
    timeZoneId.value = data.timeZoneId;
    timeZoneName.value = data.timeZoneName;
    if (hours > 12) {
      timePeriod.value = 'PM';
    } else if (hours < 12) {
      timePeriod.value = 'AM';
    }
    let hh = hours * 30;
    let mm = minutes * deg;
    let ss = seconds * deg;
    const hr = document.querySelector('#hr');
    const mn = document.querySelector('#mn');
    const sc = document.querySelector('#sc');
    hr.style.transform = `rotateZ(${hh + mm / 12}deg)`;
    mn.style.transform = `rotateZ(${mm}deg)`;
    sc.style.transform = `rotateZ(${ss}deg)`;
  });
});

onUnmounted(() => {
  clearInterval(intervalId);
});
</script>

<template>
  <div className="card w-96 bg-base-100 shadow-xl">
    <figure className="px-10 pt-10">
      <div class="ClockBody">
        <div class="clockContainer">
          <div class="clockBox">
            <div class="clockClock">
              <div class="clockHour">
                <div class="clockHr" id="hr"></div>
              </div>
              <div class="clockMin">
                <div class="clockMn" id="mn"></div>
              </div>
              <div class="clockSec">
                <div class="clockSc" id="sc"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </figure>
    <div className="card-body items-center text-center">
      <h2 className="card-title">{{ timePeriod }}</h2>
      <p>TimeZoneId: {{ timeZoneId }}</p>
      <p>TimeZoneName: {{ timeZoneName }}</p>
    </div>
  </div>
</template>

<style scoped>
.ClockBody {
  display: flex;
  justify-content: center;
  align-items: center;
}

.clockBox {
  position: relative;
  z-index: 1;
  width: 200px;
  height: 200px;
  background-color: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(25px);
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.clockClock {
  position: absolute;
  top: 10px;
  left: 10px;
  right: 10px;
  bottom: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: radial-gradient(transparent, rgba(255, 255, 255, 0.2)),
    url('../assets/images/clock.png');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  border-radius: 50%;
  backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-bottom: none;
  border-right: none;
  box-shadow: -10px -10px 20px rgba(255, 255, 255, 0.1),
    10px 10px 20px rgba(0, 0, 0, 0.1), 0px 20px 30px rgba(0, 0, 0, 0.1);
}

.clockClock::before {
  content: '';
  position: absolute;
  width: 7px;
  height: 7px;
  background: #fff;
  border-radius: 50%;
  z-index: 100;
}

.clockHour,
.clockMin,
.clockSec {
  position: absolute;
}

.clockHour,
.clockHr {
  width: 80px;
  height: 80px;
}
.clockMin,
.clockMn {
  width: 100px;
  height: 100px;
}
.clockSec,
.clockSc {
  width: 120px;
  height: 120px;
}

.clockHr,
.clockMn,
.clockSc {
  display: flex;
  flex-direction: row;
  justify-content: center;
  position: absolute;
  border-radius: 50%;
}

.clockHr::before {
  content: '';
  position: absolute;
  width: 5px;
  height: 40px;
  background-color: #ff105e;
  z-index: 11;
  border-radius: 6px;
}

.clockMn::before {
  content: '';
  position: absolute;
  width: 3px;
  height: 50px;
  background-color: #fff;
  z-index: 12;
  border-radius: 6px;
}

.clockSc::before {
  content: '';
  position: absolute;
  width: 1px;
  height: 80px;
  background-color: #fff;
  z-index: 12;
  border-radius: 6px;
}
</style>
