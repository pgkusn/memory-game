<template>
    <div class="infos fadeIn animated">
        <h2>Project 4.</h2>
        <h1>Memory<br>Blocks</h1>
        <h3 class="status">
            <template v-if="statusText">
                {{ statusText }}
            </template>
            <template v-else>
                Level {{ currentLevel }}
            </template>
        </h3>
    </div>
    <div class="blocks fadeIn animated">
        <div class="row">
            <div class="block block1" :class="{ active: isActive[0] }" @click="clickHandler(0)">
                1
            </div>
            <div class="block block2" :class="{ active: isActive[1] }" @click="clickHandler(1)">
                2
            </div>
        </div>
        <div class="row">
            <div class="block block3" :class="{ active: isActive[2] }" @click="clickHandler(2)">
                3
            </div>
            <div class="block block4" :class="{ active: isActive[3] }" @click="clickHandler(3)">
                4
            </div>
        </div>
        <div class="row">
            <div :class="['inputStatus', statusClass]">
                <div v-for="(i, index) in currentLevelData.length" :key="i" :class="['circle', { correct: index < clicked.length }]" />
            </div>
        </div>
    </div>
</template>

<script>
import { watch, computed, onMounted, ref } from 'vue';
import axios from 'axios';

export default {
    setup () {
        const currentLevel = ref(0);
        const levelData = ref([]);
        const currentLevelData = computed(() => levelData.value[currentLevel.value] || 0);
        const isActive = ref([false, false, false, false]);
        const isActiveAll = ref(false);
        const clicked = ref([]);
        const passCount = ref(0);
        const clickDisabled = ref(false);
        const status = ref(0); // 1: correct 2: wrong
        const statusText = computed(() => status.value === 1 ? 'Correct!' : status.value === 2 ? 'Wrong...' : '');
        const statusClass = computed(() => status.value === 1 ? 'correct' : status.value === 2 ? 'wrong' : '');

        // init sound
        const scales = [1, 2, 3, 4, 5, 5.5, 7, 8];
        const sounds = scales.map(scale => {
            const audioObj = new Audio(`http://awiclass.monoame.com/pianosound/set/${scale}.wav`);
            audioObj.setAttribute('preload', 'auto');
            return { scale, audioObj };
        });

        watch(isActive.value, arr => {
            if (isActiveAll.value) return;
            arr.forEach((item, index) => {
                if (item) {
                    playSound(index + 1);
                    setTimeout(() => {
                        isActive.value[index] = false;
                    }, 50);
                }
            });
        });

        const playSound = scale => {
            const sound = sounds.find(sound => sound.scale === scale);
            sound.audioObj.currentTime = 0;
            sound.audioObj.play();
        };

        const clickHandler = index => {
            if (clickDisabled.value) return;
            isActive.value[index] = true;
            clicked.value.push(index + 1);

            // check clicked value
            setTimeout(() => {
                const pass = currentLevelData.value.startsWith(clicked.value.join(''));
                if (pass) {
                    passCount.value++;
                    if (passCount.value === currentLevelData.value.length) {
                        next(true);
                    }
                }
                else {
                    next(false);
                }
            }, 50);
        };

        const demoLevel = () => {
            clickDisabled.value = true;
            const currentLevelDataArr = currentLevelData.value.split('');
            for (let i = 0; i < currentLevelDataArr.length; i++) {
                setTimeout(() => {
                    const activeIndex = currentLevelDataArr[i] - 1;
                    isActive.value[activeIndex] = true;
                    if (i === currentLevelDataArr.length - 1) {
                        clickDisabled.value = false;
                    }
                }, 500 * (i + 1));
            }
        };

        const next = async allPass => {
            if (allPass) {
                status.value = 1;
                await activeAll(allPass);
                await inactiveAll();
                currentLevel.value++;
            }
            else {
                status.value = 2;
                await activeAll(allPass);
                await inactiveAll();
                currentLevel.value = 0;
            }
            status.value = 0;
            clicked.value.length = 0;
            passCount.value = 0;
            demoLevel();
        };

        const activeAll = allPass => {
            return new Promise(resolve => {
                isActiveAll.value = true;
                clickDisabled.value = true;
                setTimeout(() => {
                    isActive.value.forEach((item, index) => {
                        isActive.value[index] = true;
                    });
                    if (allPass) {
                        playSound(1);
                        playSound(3);
                        playSound(5);
                        playSound(8);
                    }
                    else {
                        playSound(2);
                        playSound(4);
                        playSound(5.5);
                        playSound(7);
                    }
                    resolve();
                }, 500);
            });
        };

        const inactiveAll = () => {
            return new Promise(resolve => {
                setTimeout(() => {
                    isActive.value.forEach((item, index) => {
                        isActive.value[index] = false;
                    });
                    isActiveAll.value = false;
                    clickDisabled.value = false;
                    resolve();
                }, 1000);
            });
        };

        const getLevelData = async () => {
            try {
                const { data } = await axios('https://2017.awiclass.monoame.com/api/demo/memorygame/leveldata');
                levelData.value = data;
                return data;
            }
            catch (error) {
                console.error(error.message);
            }
        };

        onMounted(async () => {
            const data = await getLevelData();
            if (data) {
                demoLevel();
            }
        });

        return {
            clickHandler,
            isActive,
            currentLevel,
            currentLevelData,
            clicked,
            statusText,
            statusClass
        };
    }
};
</script>

<style lang="scss">
$colorRed: #FF5353;
$colorYellow: #FFC429;
$colorBlue: #5980C1;
$colorWhite: #FBE9B7;

@mixin size($w, $h: $w) {
  width: $w;
  height: $h;
}

@import url("https://fonts.googleapis.com/css?family=Roboto:700");

html, body {
  font-family: "roboto";
  background-color: #232526;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;

  @include size(100%);

  margin: 0;
  padding: 0;
  border: solid 5px rgba(white, 0.05);
  transition: 0.5s;
  box-sizing: border-box;

  &:active {
    transition: 0s;
    border: solid 5px rgba(white, 0.1);
  }
}

.infos {
  color: white;
  position: absolute;
  left: 40px;
  top: 40px;
  z-index: 100;

  h1 {
    margin-top: 0;
    margin-bottom: 0;
    font-size: 40px;
    letter-spacing: 1px;
    line-height: 1.2;
  }

  h3 {
    font-weight: 500;
    color: $colorRed;
  }
}

.blocks {
  .row {
    display: flex;
  }

  .block {
    animation: comeOut 1s;
    width: 100px;
    height: 100px;
    border: solid 1px rgba(white, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
    transition-duration: 0.6s;
    tranisiton-delay: 0.2s;
    cursor: pointer;
    margin: 12px;
    color: white;
    font-weight: 100;

    @mixin setBlockColor($color) {
      box-shadow: 0px 0px 35px rgba($color, 0.2) inset, 0px 0px 30px rgba($color, 0.2);
      background-color: transparent;
      border: solid 1px lighten($color, 5);
      color: transparent;

      &:hover {
        background-color: rgba($color, 0.3);
        box-shadow: 0px 0px 35px rgba($color, 0.2);
      }

      &.active {
        background-color: $color;
        box-shadow: 0px 0px 35px $color;

        // color: white
        // transform: scale(0.95)

      }
    }

    &.active {
      transition-duration: 0s;
    }

    &.block1 {
      @include setBlockColor($colorRed);
    }

    &.block2 {
      @include setBlockColor($colorYellow);
    }

    &.block3 {
      @include setBlockColor($colorBlue);
    }

    &.block4 {
      @include setBlockColor($colorWhite);
    }
  }
}

.inputStatus {
  margin-top: 10px;

  .circle {
    @include size(6px);

    display: inline-block;
    opacity: 0.3;
    background-color: white;
    border-radius: 100%;
    margin: 5px;

    &.correct, .wrong {
      opacity: 1;
    }
  }

  &.wrong .circle {
    background-color: $colorRed;
    opacity: 1;
  }

  &.correct .circle {
    background-color: $colorBlue;
    opacity: 1;
  }
}
</style>
