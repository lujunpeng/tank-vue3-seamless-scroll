<template>

  <div ref="ref_tank_seamless_scroll" class="tank_seamless_scroll" :style="{height: parentHeight+'px'}">
    <div class="debugger" v-if="debug">
      copyCount:{{ loopCount.length }}
      <br/>
      translateY:{{ Math.round(val * 1000) / 1000 }}px
      <br/>
      fps:{{ currentFps }}
    </div>
    <div ref="ref_warp" class="warp">
      <template v-for="i in loopCount.length" :key="'ss_'+i">
        <keep-alive>
          <div ref="ref_warpLine" class="warpLine" v-if="!hideItemsIndex.includes(i)">
            <slot name="default"></slot>
          </div>
        </keep-alive>
      </template>
    </div>
  </div>
</template>

<script setup>
import {onMounted, onUnmounted, onBeforeUnmount, nextTick, getCurrentInstance, ref} from "vue";
import {timer} from "d3-timer";

const loopCount = ref(new Array(5))
const prop = defineProps({
  stepLength: {
    type: Number,
    default: 60,
    validator: (value) => value >= 0,
  },
  reverse: {
    type: Boolean,
    default: false
  },
  debug: {
    type: Boolean,
    default: false
  },
  limitFps: {
    type: Number,
    default: 32
  },
  pauseOnHover: {
    type: Boolean,
    default: true
  },
})
let t = null
let tWatch = null
const ref_tank_seamless_scroll = ref(null)
const ref_warpLine = ref(null)
const ref_warp = ref(null)
let parentHeight = ref(0)
const createTimer = (handle, fps = 32) => new Promise((resolve, reject) => {
  resolve(timer(() => {
    try {
      handle()
    } catch (err) {
      reject(err)
    }
  }, 1000 / fps))
})
const getTime = () => new Date().getTime()
const val = ref()
const currentFps = ref(60)
let animation = true
let lastTime = getTime()
const onStop = (evt) => animation = false
const onPlay = (evt) => {
  animation = true
  lastTime = getTime()
}
const hideItemsIndex = ref([])
onMounted(() => {

      nextTick(() => {

        ref_tank_seamless_scroll.value.addEventListener("mouseover", onStop)
        ref_tank_seamless_scroll.value.addEventListener("mouseout", onPlay)
        const parentElementRect = ref_tank_seamless_scroll.value.parentElement.getBoundingClientRect()
        parentHeight.value = parentElementRect.height
        let warpLineHeight = ref_warpLine.value[0].getBoundingClientRect().height;
        let lastY = -warpLineHeight;
        let translateY = -warpLineHeight
        loopCount.value = new Array(Math.ceil(parentHeight.value / warpLineHeight) * 3)
        createTimer(() => {
          if (!animation && prop.pauseOnHover) {
            return;
          }
          const limitHeight = prop.reverse ? -warpLineHeight * (loopCount.value.length / 3) : -warpLineHeight * (loopCount.value.length / 3 * 2)
          const currentTime = getTime()
          currentFps.value = Math.ceil(1000 / (currentTime - lastTime))
          const len = (currentTime - lastTime) * (prop.stepLength / 1000)
          lastTime = currentTime
          translateY += (prop.reverse ? 1 : -1) * len
          if (prop.reverse) {
            translateY = translateY > limitHeight ? -(warpLineHeight + Math.abs(translateY % warpLineHeight)) : translateY
            //-(warpLineHeight + Math.abs(translateY % warpLineHeight)) : translateY

          } else {
            translateY = translateY < limitHeight ?
                Math.sign(translateY) * (Math.abs(translateY % warpLineHeight) + warpLineHeight) :
                translateY
          }
          val.value = translateY
          ref_warp.value.style.setProperty("transform", `translate3d(0px ,${translateY}px,0px)`)
          lastY = translateY
          const hideItemsIndex = []


          //用for循环遍历判断ref_warpLine.value，数组中的元素是否在可视区域内，只判断x和y
          ref_warpLine.value.forEach((item, i) => {
            let currenRect = ref_warpLine.value[i].getBoundingClientRect()
            let parentElementRect = ref_tank_seamless_scroll.value.parentElement.getBoundingClientRect()
            if (currenRect.y + currenRect.height < parentElementRect.y || currenRect.y > parentElementRect.y + parentElementRect.height) {
              hideItemsIndex.push(i)
            } else {
              hideItemsIndex.splice(hideItemsIndex.value.indexOf(i), 1)
            }
          })

          hideItemsIndex.value = hideItemsIndex;


        }).then((timer) => {
          t = timer
        }).catch(err => {
          console.error(err)
        })
        createTimer(() => {
          parentHeight.value = ref_tank_seamless_scroll.value.parentElement.getBoundingClientRect().height
          warpLineHeight = ref_warpLine.value[0].getBoundingClientRect().height;
          loopCount.value = new Array(Math.ceil(parentHeight.value / warpLineHeight) * 3)
        }).then((timer) => {
          tWatch = timer
        }).catch(err => {
          console.error(err)
        })
      })
    }
)
onBeforeUnmount(() => {
  t?.stop()
  tWatch?.stop()
  if (ref_tank_seamless_scroll.value) {
    ref_tank_seamless_scroll.value.removeEventListener("mouseover", onStop)
    ref_tank_seamless_scroll.value.removeEventListener("mouseout", onPlay)
  }
})
</script>

<style scoped>
.tank_seamless_scroll {
  position: relative;
  overflow: hidden;
}

.tank_seamless_scroll, .warpLine, .warp {
  padding: 0;
  margin: 0;
  transition: 0s;
  overflow: hidden;
  /*border-bottom: transparent 1px solid; !* key code *!*/
  /*box-sizing: border-box;*/
}

.warpLine {

}

.debugger {
  position: relative;
  top: 5px;
  left: 10.5%;
  width: 75%;
  height: auto;
  padding: 5px 2.5%;
  background-color: rgba(0, 0, 0, .8);
  color: #999999;
  z-index: 88;
  border-radius: 10px;
  border: solid 1px #999999;
}

</style>
