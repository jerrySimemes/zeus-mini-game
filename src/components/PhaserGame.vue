<template>
  <!-- 預載入圖片後遮罩消失，目的是蓋住 Game Start 畫面 -->
  <transition leave-active-class="transition-opacity duration-100 ease-in" leave-from-class="opacity-100" leave-to-class="opacity-0">
    <div v-if="!$store.isLoaded" class="absolute w-full h-full bg-black z-10"></div>
  </transition>
  <div>
    <!-- UI -->
    <div
      class="absolute translate-x-[calc(50vw-50.1%)] translate-y-[calc(50vh-50%)] inset-0 aspect-[720/1280] max-w-full max-h-full z-[1] pointer-events-none"
    >
      <!-- 遮罩，startPanel & pausePanel 都在遊戲上層，遊戲層可能邊緣會露出 -->
      <div v-if="$store.isPaused || !$store.isStart" class="absolute bg-black w-[100.1%] h-full z-10"></div>
      <!-- Score -->
      <div
        class="absolute top-1 w-full h-[100px]"
        :style="{ maxWidth:  + 'px' }"
      >
        <div class="relative">
          <!-- 底 -->
          <div class="w-[95%] h-8 left-[2.5%] rounded-[20px] bg-[#C87637] border-[2px] border-black absolute top-4"></div>
          <!-- logo -->
          <img src="/images/simemes_logo.png" class="w-25 absolute top-2 left-4" />
          <!-- 分數區 -->
          <div class="w-[37%] h-15 rounded-[13px] bg-[#C87637] border-[2px] border-black absolute top-0 left-[31.5%] flex justify-center items-center">
            <div class="w-[92%] h-[80%] rounded-[10px] bg-[#643B1B] text-center text-[32px] font-[Impact]">
              {{$store.totalScore}}
            </div>
          </div>
          <!-- 暫停按鈕區 -->
          <div class="w-12 h-12 rounded-[10px] bg-[#C87637] border-[2px] border-black absolute top-2 right-5 flex justify-center items-center">
            <img
              @click="togglePause"
              src="/images/pause_btn.png"
              class="w-[90%] h-[90%] top-2 left-4 pointer-events-auto" 
            />
          </div>
          <!-- 加時提示 -->
          <transition-group name="tip" tag="div" class="absolute top-5 z-20 flex flex-col w-[30%]"
            enter-active-class="transition duration-300 ease-out"
            enter-from-class="opacity-0 translate-y-4"
            enter-to-class="opacity-100 translate-y-0"
            leave-active-class="transition duration-300 ease-in"
            leave-from-class="opacity-100 translate-y-0"
            leave-to-class="opacity-0 -translate-y-4"
            >
            <div
              v-for="tip in timeTips"
              :key="tip.id"
              class="relative top-20 text-white text-[16px]text-[16px] sm:text-[20px] md:text-[30px] lg:text-[40px] xl:text-[50px] font-bold px-2 py-1 rounded w-[100%] font-[Impact] [text-shadow:1px_1px_0_#000,-1px_-1px_0_#000,1px_-1px_0_#000,-1px_1px_0_#000] my-[2px]"
            >
              +{{ tip.value }} SECONDS
            </div>
          </transition-group>
        </div>
      </div>
      <!-- Start -->
      <div
        v-if="(4 > sec && sec >= 0) || clockSec == 0"
        class="absolute top-[40%] w-full text-[100px] leading-[100px] tracking-[0%] text-center font-[Impact]"
        :style="{ maxWidth:  + 'px' }"
      >
        {{countdownSec}}
        <div
          v-if="(3 > sec)"
          class="w-[80%] flex mt-20 mx-auto">
          <img src="/images/arrow_l.png" class="w-[50%]">
          <img src="/images/arrow_r.png" class="w-[50%]">
        </div>
      </div>
      <!-- Stage2 Hint -->
      <div
        v-if="clockSec == 40 && !hasStage2"
        class="absolute top-[40%] w-full text-[100px] leading-[100px] tracking-[0%] text-center font-[Impact]"
        :style="{ maxWidth:  + 'px' }"
      >
        STAGE - 2
      </div>
      <!-- Stage3 Hint -->
      <div
        v-if="clockSec == 20 && !hasStage3"
        class="absolute top-[40%] w-full text-[100px] leading-[100px] tracking-[0%] text-center font-[Impact]"
        :style="{ maxWidth:  + 'px' }"
      >
        STAGE - 3
      </div>
      <!-- Time -->
      <div
        class="absolute bottom-0 w-full"
        :style="{ maxWidth:  + 'px' }"
      >
        <div class="relative flex aspect-[10]">
          <!-- bar bg -->
          <img src="/images/time_bar.png" class="absolute bottom-0">
          <!-- clock & progress 的容器 -->
          <div class="relative w-full flex px-2">
            <!-- clock bg -->
            <div class=" w-[45px] h-[22px] py-[2px] mr-3 bg-[#643B1B] rounded-[20px] relative flex justify-start">
              <!-- icon -->
              <img src="/images/clock_icon.png" class="relative w-[16px] h-[16px] mx-1">
              <!-- sec -->
              <p class="sec-font relative text-[18px] leading-[100%]">{{clockSec}}</p>
            </div>
            <!-- progress bar -->
            <div class="flex-1 h-[15px] bg-[#643B1B] mt-1 rounded-[20px] relative left-0 z-1 overflow-hidden">
              <!-- 實質時間 -->
              <div class="h-[15px] bg-[#FFDB34] rounded-[20px] relative"
                :style="{ width: (clockSec / 60 * 100) + '%' }"></div>
              <!-- 三小格 -->
              <div class="h-[15px] bottom-0 border border-[#643B1B] absolute w-[33.3%] rounded-tl-[20px] rounded-bl-[20px] rounded-tr-[1px] rounded-br-[1px]"></div>
              <div class="h-[15px] bottom-0 border-y border-[#643B1B] absolute w-[33.3%] rounded-tl-[1px] rounded-bl-[1px] rounded-tr-[1px] rounded-br-[1px] left-[33.3%]"></div>
              <div class="h-[15px] bottom-0 border border-[#643B1B] absolute w-[33.3%] rounded-tl-[1px] rounded-bl-[1px] rounded-tr-[20px] rounded-br-[20px] left-[66.6%]"></div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- Canvas -->
    <div
      ref="gameContainer"
      class="relative w-screen h-screen mx-auto"
    >
    </div>
  </div>

  <div
    class="absolute translate-x-[calc(50vw-50%)] translate-y-[calc(50vh-50%)] inset-0 aspect-[720/1280] max-w-full max-h-full z-[1] pointer-events-none overflow-hidden"
  >
    <div v-if="!$store.isLoaded || $store.isLoadPage" class="absolute top-0 z-3 w-full h-full flex flex-col justify-center">
      <LoadPage></LoadPage>
    </div>
    <div v-if="!$store.isStart" class="absolute top-0 z-2 w-full h-full flex flex-col justify-center">
      <Start @startEvent = "activeGameStart"></Start>
    </div>
    <div v-if="$store.isPaused" class="absolute top-0 z-1 w-full h-full flex flex-col justify-center">
      <Pause @pauseEvent = "togglePause"></Pause>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, onBeforeUnmount, ref, computed } from "vue";
import Phaser from "phaser";
import { useStore } from '../stores/store'
import Pause from '../components/Pause.vue'
import Start from '../components/Start.vue'
import LoadPage from '../components/LoadPage.vue'
// import { useRouter } from 'vue-router'
// const router = useRouter()
const $store = useStore()

const gameContainer = ref<HTMLDivElement | null>(null);
// 觸碰位置與玩家位置間，不移動的緩衝區間
const pointerDeadZone = 10;
// 玩家最大速度
const playerMaxSpeed = 10000;
// 速度係數，數字越小，速度變化越慢
const inputScale = 25;
const timeTips = ref<{ id: number; value: number }[]>([]);
// 預載入圖片
const imageList: string[] = [
  './images/bg_blue_sky.jpg',
  './images/arrow_l.png',
  './images/arrow_r.png',
  './images/bomb.png',
  './images/clock.png',
  './images/clock_gold.png',
  './images/clock_icon.png',
  './images/coin.png',
  './images/gmove.png',
  './images/hat.png',
  './images/invincible.png',
  './images/knockout.png',
  './images/pause_btn.png',
  './images/pepe_in_chest.png',
  './images/player.png',
  './images/poseidon.png',
  './images/simemes_bg.png',
  './images/simemes_logo.png',
  './images/smoke.png',
  './images/star.png',
  './images/thunder.png',
  './images/time_bar.png',
  './images/zeus_drop_logo.png',
  './images/zeus.png',
];
// Stage 1、2、3 itemList settings
const itemList1 = [
  // 得分 - weight 大
  { key: 'gmove', scale: 0.15, speed: [200, 900], weight: 5, scores: 100, delay: 0, plus_time: 0 },
  { key: 'hat', scale: 0.15, speed: [200, 900], weight: 5, scores: 150, delay: 0, plus_time: 0 },
  { key: 'coin', scale: 0.15, speed: [200, 900], weight: 5, scores: 500, delay: 0, plus_time: 0 },
  // 加時 - weight 中
  { key: 'clock', scale: 0.15, speed: [200, 900], weight: 1, scores: 0, delay: 0, plus_time: 2 },
  { key: 'clock_gold', scale: 0.15, speed: [200, 900], weight: 1, scores: 0, delay: 0, plus_time: 5 },
  // 暈眩 - weight 中
  { key: 'bomb', scale: 0.15, speed: [200, 900], weight: 4, scores: 0, delay: 2, plus_time: 0 },
  // 扣分
  { key: 'thunder', scale: 0.15, speed: [200, 900], weight: 5, scores: 200, delay: 0, plus_time: 0 }
];
const itemList2 = [
  // 得分 - weight 大
  { key: 'gmove', scale: 0.15, speed: [600, 1300], weight: 5, scores: 200, delay: 0, plus_time: 0 },
  { key: 'hat', scale: 0.15, speed: [600, 1300], weight: 5, scores: 300, delay: 0, plus_time: 0 },
  { key: 'coin', scale: 0.15, speed: [600, 1300], weight: 5, scores: 1000, delay: 0, plus_time: 0 },
  // 加時 - weight 中
  { key: 'clock', scale: 0.15, speed: [600, 1300], weight: 1, scores: 0, delay: 0, plus_time: 2 },
  { key: 'clock_gold', scale: 0.15, speed: [600, 1300], weight: 1, scores: 0, delay: 0, plus_time: 5 },
  // 暈眩 - weight 中
  { key: 'bomb', scale: 0.15, speed: [600, 1300], weight: 10, scores: 0, delay: 2, plus_time: 0 },
  // 扣分
  { key: 'thunder', scale: 0.15, speed: [600, 1300], weight: 5, scores: 400, delay: 0, plus_time: 0 },
];
const itemList3 = [
  // 得分 - weight 大
  { key: 'gmove', scale: 0.15, speed: [900, 2500], weight: 5, scores: 300, delay: 0, plus_time: 0 },
  { key: 'hat', scale: 0.15, speed: [900, 2500], weight: 5, scores: 450, delay: 0, plus_time: 0 },
  { key: 'coin', scale: 0.15, speed: [900, 2500], weight: 5, scores: 1500, delay: 0, plus_time: 0 },
  // 加時 - weight 中
  { key: 'clock', scale: 0.15, speed: [900, 2500], weight: 1, scores: 0, delay: 0, plus_time: 2 },
  { key: 'clock_gold', scale: 0.15, speed: [900, 2500], weight: 1, scores: 0, delay: 0, plus_time: 5 },
  // 暈眩 - weight 中
  { key: 'bomb', scale: 0.15, speed: [900, 2500], weight: 20, scores: 0, delay: 2, plus_time: 0 },
  // 扣分
  { key: 'thunder', scale: 0.15, speed: [900, 2500], weight: 5, scores: 600, delay: 0, plus_time: 0 },
];

let game: Phaser.Game | null = null;
let resultTimeout = ref<ReturnType<typeof setTimeout> | null>(null);
let timerEvent = ref<Phaser.Time.TimerEvent | null>(null);
let hasStage2 = false;
let hasStage3 = false;
let gameStart = ref(false)
let sec = ref(0)
let clockSec = ref(60)
let pointerX: number | null = null;
let itemList = [
  // 得分 - weight 大
  { key: 'gmove', scale: 0.15, speed: [200, 900], weight: 5, scores: 100, delay: 0, plus_time: 0 },
  // { key: 'hat', scale: 0.15, speed: [200, 900], weight: 5, scores: 150, delay: 0, plus_time: 0 },
  // { key: 'thunder', scale: 0.15, speed: [200, 900], weight: 5, scores: 200, delay: 0, plus_time: 0 },
  // { key: 'poseidon', scale: 0.15, speed: [200, 900], weight: 5, scores: 300, delay: 0, plus_time: 0 },
  // { key: 'coin', scale: 0.15, speed: [200, 900], weight: 5, scores: 500, delay: 0, plus_time: 0 },
  // // 加時 - weight 中
  // { key: 'clock', scale: 0.15, speed: [200, 900], weight: 3, scores: 0, delay: 0, plus_time: 2 },
  // { key: 'clock_gold', scale: 0.15, speed: [200, 900], weight: 3, scores: 0, delay: 0, plus_time: 5 },
  // // 暈眩 - weight 中
  // { key: 'bomb', scale: 0.15, speed: [200, 900], weight: 3, scores: 0, delay: 2, plus_time: 0 },
  // // 無敵 - weight 小
  // { key: 'star', scale: 0.15, speed: [200, 900], weight: 1, scores: 0, delay: 0, plus_time: 0 },
];

let boss: Phaser.GameObjects.Sprite;
let player: Phaser.Physics.Arcade.Sprite;
let items: Phaser.Physics.Arcade.Group;
let b_direction = Math.random() < 0.5 ? -1 : 1; // 初始方向
let b_speed = Phaser.Math.Between(2, 6); // 初始速度 2~6
let b_changeDirCooldown = 0;
let hasStarted = false;
let isTouching = false;
let hasGotoResult = false

let timeTipId = 0;

// ================================== function ==================================

// ------------- 預載入圖片 -------------
function preloadImages(imageUrls: string[]) {
  console.log("[zeus]: preloadImages from Home ...")
  return Promise.all(
    imageUrls.map(
      (src) =>
        new Promise((resolve, reject) => {
          const img = new Image();
          img.src = src;
          img.onload = resolve;
          img.onerror = reject;
        })
    )
  );
}

// ------------- 背景響應式調整 -------------
function fitBackground(bg: Phaser.GameObjects.Image, scene: Phaser.Scene) {
  const { width, height } = scene.scale;
  const scale = Math.max(width / bg.width, height / bg.height);
  bg.setScale(scale);
}

// ------------- 開始遊戲按鈕 -------------
function activeGameStart() {
  $store.isStart = true
  StartCountdown();
}

// ------------- 預備三秒後啟動 -------------
function StartCountdown() {
  const interval = setInterval(() => {
    sec.value++;
    if (sec.value === 3) {
      gameStart.value = true;
      StartClock();
    }
    if (sec.value === 4) clearInterval(interval);
  }, 1000);
}

// ------------- 時鐘開始倒數 -------------
function StartClock() {
  console.log("StartClock!")
  const interval = setInterval(() => {
    if (!$store.isPaused) {
      clockSec.value--;
      // STAGE 2
      if (clockSec.value === 40) {
        $store.stage = 2
        console.log('STAGE 2!')
      }
      // STAGE 3
      if (clockSec.value === 20) {
        $store.stage = 3
        console.log('STAGE 3!')
      }
      // 時間結束
      if (clockSec.value === 0) {
        console.log('Time\'s Up!')
        clearInterval(interval);
      }
    }
  }, 1000);
}

// ------------- 暫停按鈕 -------------
function togglePause() {
  if(gameStart.value && clockSec.value != 0) {
    $store.isPaused = !$store.isPaused;
    // 確保遊戲存在且目前場景是活躍狀態
    if (game && game.scene.isActive('default')) {
      const scene = game.scene.getScene('default');
      // 暫停狀態
      if ($store.isPaused) {
        console.log('Pause')
        scene.physics.world.pause();
        scene.time.timeScale = 0;
      // 非暫停狀態
      } else {
        console.log('Continue')
        scene.physics.world.resume();
        scene.time.timeScale = 1;
      }
    }
  }
}

// ------------- 隨機掉落物品 -------------
function dropRandomItem(x: number, y: number) {
  // 依照 weight 建立擴展陣列
  const weightedList: string[] = []
  if($store.stage == 1) {
    itemList = itemList1
  } else if($store.stage == 2) {
    itemList = itemList2
  } else {
    itemList = itemList3
  }
  itemList.forEach(item => {
    for (let i = 0; i < item.weight; i++) {
      weightedList.push(item.key)
    }
  })
  const selectedKey = Phaser.Utils.Array.GetRandom(weightedList)
  const itemData = itemList.find(i => i.key === selectedKey)
  if (!itemData) return

  const item = items.create(x, y, selectedKey) as Phaser.Physics.Arcade.Sprite
  const randomSpeed = Phaser.Math.Between(itemData.speed[0], itemData.speed[1]);
  item.setVelocityY(randomSpeed)
  item.setScale(itemData.scale)
  item.setData('type', selectedKey) // 方便之後判斷
}

// ----------- 開始定時丟東西 -----------
function droppingItems(scene: Phaser.Scene) {
  timerEvent.value = scene.time.addEvent({
    delay: $store.stage == 3 ? 300 : $store.stage == 2 ? 600 : 900,
    loop: true,
    callback: () => {
      dropRandomItem(boss.x, boss.y + 50);
    },
  });
}

// ----------- 取消定時丟東西 -----------
function stopDroppingItems() {
  if (timerEvent.value) {
    timerEvent.value.remove(false);
    timerEvent.value = null;
  }
}

// ----------- bomb_smoke_anim -----------
function smokeAnim(scene: Phaser.Scene) {
  // play anim
  const smoke = scene.add.sprite(player.x, player.y - 180, 'bomb_smoke', 'Smoke_00000.png');
  smoke.play('bomb_smoke_anim');
  // 播放完畢後移除
  smoke.on('animationcomplete', () => {
    smoke.destroy();
  });
}

// ----------- 顯示加時提示 -----------
function showTimeTip(amount: number) {
  const id = timeTipId++;
  timeTips.value.push({ id, value: amount });

  setTimeout(() => {
    timeTips.value = timeTips.value.filter((t) => t.id !== id);
  }, 1500); // 1.5 秒後移除提示
}

// ----------- 顯示加分提示 -----------
function showScoreTip(scene: Phaser.Scene, x: number, y: number, text: string) {
  // 判斷文字顏色：+ 白色，- 紅色
  const color = text.startsWith('+') ? '#ffffff' : text.startsWith('-') ? '#ff4444' : '#ffffff';
  const scoreText = scene.add.text(x, y, text, {
    fontFamily: 'Impact',
    fontSize: '40px',
    color: color,
    stroke: '#000000',
    strokeThickness: 4,
  }).setOrigin(0.5)

  scene.tweens.add({
    targets: scoreText,
    y: y - 200,
    alpha: 0,
    duration: 800,
    ease: 'ease.out',
    onComplete: () => {
      scoreText.destroy()
    }
  });
}

// ------------- 跳去 result 頁面 -------------
function GotoResult() {
  resultTimeout.value = setTimeout(() => {
    // router.push('/result')
    $store.isResult = true
  }, 3000);
}

// ================================== computed ==================================

const countdownSec = computed(() => {
  let result;
  if(sec.value == 3) result = 'START';
  if(clockSec.value == 0) result = 'TIME\'S UP'
  return result?result:3-sec.value
})

// ================================== onMounted ==================================

onMounted(async() => {
  // 預載入圖片
  await preloadImages(imageList);
  $store.isLoaded = true; 

  if (!gameContainer.value) return;

  const config: Phaser.Types.Core.GameConfig = {
    type: Phaser.AUTO,
    width: 720,
    height: 1280,
    parent: gameContainer.value,
    backgroundColor: "#000000",
    physics: {
      default: "arcade",
      arcade: {
        // gravity: { x: 0, y: 0 },
        debug: false,
      },
    },
    scale: {
      mode: Phaser.Scale.FIT,
      autoCenter: Phaser.Scale.CENTER_BOTH,
    },
    scene: {
      preload,
      create,
      update,
    },
  };

  game = new Phaser.Game(config);
  // console.log('🟢 ', game.canvas)

  // -------------------------- *** preload *** --------------------------
  function preload(this: Phaser.Scene) {
    // bg
    this.load.image("bg", "./images/bg_blue_sky.jpg");
    // char
    this.load.image("boss", "./images/zeus.png");
    this.load.image("player", "./images/player.png");
    this.load.image("invincible", "./images/invincible.png");
    this.load.image("knockout", "./images/knockout.png");
    // item
    this.load.image("bomb", "./images/bomb.png");
    this.load.image("clock", "./images/clock.png");
    this.load.image("clock_gold", "./images/clock_gold.png");
    this.load.image("coin", "./images/coin.png");
    this.load.image("star", "./images/star.png");
    this.load.image("gmove", "./images/gmove.png");
    this.load.image("hat", "./images/hat.png");
    this.load.image("poseidon", "./images/poseidon.png");
    this.load.image("thunder", "./images/thunder.png");
    // atlas
    this.load.atlas("bomb_smoke", "./images/bomb_smoke.png", "./images/bomb_smoke.json")
  }

  // -------------------------- *** create *** --------------------------
  function create(this: Phaser.Scene) {
    
    // background
    const bg = this.add.image(0, -90, "bg").setOrigin(0);
    fitBackground(bg, this);
    // 監聽畫面縮放
    this.scale.on("resize", () => {
      fitBackground(bg, this);
    });

    // Boss
    boss = this.add.sprite(360, 250, "boss");
    boss.setScale(0.3);

    // Player
    player = this.physics.add
      .sprite(360, 1060, "player")
      .setCollideWorldBounds(true);
    player.setScale(0.4);

    // 玩家觸控控制
    this.input.on("pointerdown", (pointer: Phaser.Input.Pointer) => {
      isTouching = true;
      pointerX = pointer.x;
    });
    this.input.on("pointermove", (pointer: Phaser.Input.Pointer) => {
        pointerX = pointer.x;
    });
    this.input.on("pointerup", () => {
      isTouching = false;
      pointerX = null;
    });

    // bomb_smoke_anim
    this.anims.create({
      key: 'bomb_smoke_anim',
      frames: this.anims.generateFrameNames('bomb_smoke', {
        prefix: 'Smoke_000',
        start: 0,
        end: 43,
        suffix: '.png',
        zeroPad: 2 // 00000 -> zeroPad: 5，但你的是 000，所以填 2 就夠
      }),
      frameRate: 30, // 或 24, 60 根據需要
      repeat: 0 // 播放一次
    });

    // Items group
    items = this.physics.add.group();
    // 碰撞判定
    this.physics.add.overlap(player, items, (_, item) => {
      const gameItem = item as Phaser.GameObjects.GameObject & Phaser.Physics.Arcade.Body
      const type = (gameItem as any).getData?.('type')
      const itemInfo = itemList.find(i => i.key === type);
      // console.log(itemInfo)
      // 暈眩
      if (type === 'bomb') {
        if($store.invincible) return
        $store.knockOut = true
        let knockout_time = 0
        const interval = setInterval(() => {
          knockout_time++
          if (knockout_time === itemInfo!.delay) {
            $store.knockOut = false
            clearInterval(interval)
          }
        }, 1000);
        smokeAnim(this);
      // 加時
      } else if (['clock', 'clock_gold'].includes(type)) {
        clockSec.value += itemInfo!.plus_time
        // 顯示於 UI
        showTimeTip(itemInfo!.plus_time);
        if(clockSec.value >= $store.stageTime) clockSec.value = $store.stageTime
      // 得分
      } else if (['coin', 'gmove', 'hat', 'poseidon'].includes(type)) {
        $store.totalScore += itemInfo!.scores
        // 顯示於 UI
        showScoreTip(this, player.x, player.y - 200, '+' + itemInfo!.scores)
      // 扣分
      } else if (type === 'thunder') {
        $store.totalScore <= itemInfo!.scores ? 0 : $store.totalScore -= itemInfo!.scores
        // 顯示於 UI
        showScoreTip(this, player.x, player.y - 200, '-' + itemInfo!.scores)
      // 無敵
      } else if (type === 'star') {
        $store.invincible = true
        let invincible_time = 0
        const interval = setInterval(() => {
          invincible_time++
          if (invincible_time === 2) {
            $store.invincible = false
            clearInterval(interval)
          }
        }, 1000);

      }
      item.destroy()
    })

  }

  // -------------------------- *** update *** --------------------------
  function update(this: Phaser.Scene) {
    
    // 監控遊戲是否開始，只做一次
    if(gameStart.value && !hasStarted) {
      hasStarted = true;
      // 定時丟東西
      droppingItems(this);
    }
    // 監控遊戲是否開始，只做一次
    if( clockSec.value <= 40 && !hasStage2) {
      setTimeout(() => {
        hasStage2 = true;
      }, 1000)
      $store.stageTime = 40
      stopDroppingItems();
      // 定時丟東西
      droppingItems(this);
    }
    // 監控遊戲是否開始，只做一次
    if(clockSec.value <= 20 && !hasStage3) {
      setTimeout(() => {
        hasStage3 = true;
      }, 1000)
      $store.stageTime = 20
      stopDroppingItems();
      // 定時丟東西
      droppingItems(this);
    }

    // 處理遊戲尚未開始 or 已經結束 or 暫停
    if (!hasStarted || clockSec.value <= 0 || $store.isPaused) {
      // 設為零，不然會滑動到邊界
      player.setVelocityX(0);
      // 時間到時
      if (clockSec.value <= 0) {
        // 把碰撞關掉
        player.body!.checkCollision.none = true;
        // 若 knockout 設高度
        if (player.texture.key === 'knockout') player.setY(player.y + 50);
        // if(!$store.knockOut && !$store.invincible) player.setTexture('player')

        // 啟動一次 去 result 頁
        if (!hasGotoResult) {
          hasGotoResult = true
          GotoResult()
        }
      }
      return;
    }

    // 以下為正常遊戲時間內的邏輯
    // 魔王移動
    boss.x += b_direction * b_speed;
    b_changeDirCooldown--;
    // 邊界檢查用
    const halfWidth = boss.displayWidth / 2;
    if (b_changeDirCooldown <= 0) {
    // 每 60 幀（大約 1 秒）有機率改變方向
    if (Math.random() < 0.5) {
      // 不設 if(boss.x < 710 - halfWidth || boss.x > 10 + halfWidth) 有機會在邊界反彈卡死
      if(boss.x < 710 - halfWidth && boss.x > 10 + halfWidth) b_direction *= -1;
        b_speed = $store.stage == 3 ?Phaser.Math.Between(15, 20) : $store.stage == 2 ?Phaser.Math.Between(6, 15) : Phaser.Math.Between(2, 6) // ✅ 隨機新速度
      }
      b_changeDirCooldown = 60; // 重設冷卻
    }
    // 避免魔王走出畫面
    // 也避免速度太快出框後卡死抖動，要分開寫
    if (boss.x >= 720 - halfWidth) {
      b_direction = -1;
    } else if (boss.x <= 0 + halfWidth) {
      b_direction = 1;
    }

    if (isTouching && pointerX !== null && !$store.knockOut) {
      let dx = pointerX - player.x;
      // 加入 pointerDeadZone
      if (Math.abs(dx) < pointerDeadZone) {
        dx = 0;
      }
      const vx = Phaser.Math.Clamp(dx * inputScale, -playerMaxSpeed, playerMaxSpeed);
      player.setVelocityX(vx);
    } else {
      player.setVelocityX(0);
    }

    // knockout 關閉碰撞
    if ($store.knockOut) {
      player.body!.checkCollision.none = true;
    } else {
      player.body!.checkCollision.none = false;
    }
    // 更換圖示相關
    // 無敵
    if($store.invincible) {
      player.setTexture('invincible')
    } else if($store.knockOut) {
      player.setTexture('knockout')
      player.setY(player.y + 50);
    } else {
      player.setTexture('player')
    }

  }

});

onBeforeUnmount(() => {
  game?.destroy(true)

  if (resultTimeout.value) {
    clearTimeout(resultTimeout.value);
    resultTimeout.value = null;
  }
});
</script>
<style scoped>

.sec-font {
  font-family: Passion One;
}
</style>
