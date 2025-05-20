<template>
  <div class="eclipse-menu">
    <div class="eclipse-wrapper">
      <div class="central-eclipse">
        <div class="sun"></div>
        <div class="moon"></div>
        <div class="title">CONSOLES</div>
      </div>
      
      <div class="menu-ring">
        <div
          v-for="(item, index) in menuItems"
          :key="index"
          class="menu-option"
          :class="{ 'is-focused': selectedIndex === index }"
          :style="getItemPosition(index)"
          @click="updateSelection(index)"
        >
          <div class="option-content">
            <div class="mini-eclipse" :class="{ 'back-button': item === 'BACK' }" @click="handleItemClick(index)">
              <div class="mini-eclipse-inner"></div>
            </div>
            <span class="option-text">{{ item }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const menuItems = ['PC', 'NES', 'SNES', 'GENESIS', 'GAME BOY', 'GB COLOR', 'GBA', 'PS1', 'N64', 'BACK', 'DREAMCAST', 'DS']
const selectedIndex = ref(0)
const gamepadIndex = ref(null)
const canMove = ref(true)
const lastAButtonState = ref(false)
const isTransitioning = ref(false)

function getItemPosition(index) {
  const totalItems = menuItems.length
  const angleStep = (2 * Math.PI) / totalItems
  const radius = 250
  const angle = angleStep * index - Math.PI / 2
  
  const x = Math.cos(angle) * radius
  const y = Math.sin(angle) * radius
  
  return {
    transform: `translate(${x}px, ${y}px)`
  }
}

function updateSelection(newIndex) {
  if (newIndex >= 0 && newIndex < menuItems.length) {
    selectedIndex.value = newIndex
  }
}

function handleItemClick(index) {
  const selectedItem = menuItems[index]
  if (selectedItem === 'BACK') {
    window.location.href = '/'
  } else if (selectedItem === 'DS') {
    router.push('/games')
   } else if (selectedItem === 'PC') {
      router.push('/pc')
    } else if (selectedItem === 'nes') {
      router.push('/nes')
    } else if (selectedItem === 'snes') {
      router.push('/snes')
    } else if (selectedItem === 'genesis') {
      router.push('/genesis')
    }
  }

function gamepadConnectHandler(e) {
  gamepadIndex.value = e.gamepad.index
}

function gamepadDisconnectHandler() {
  gamepadIndex.value = null
}

function checkGamepad() {
  let animationFrameId = null

  const checkGamepadInternal = () => {
    if (gamepadIndex.value !== null) {
      const gamepad = navigator.getGamepads()[gamepadIndex.value]
      if (gamepad) {
        const axisValue = gamepad.axes[0]
        const dpadLeft = gamepad.buttons[14].pressed
        const dpadRight = gamepad.buttons[15].pressed
        const aButton = gamepad.buttons[0].pressed

        // Handle movement
        if ((Math.abs(axisValue) > 0.5 || dpadLeft || dpadRight) && canMove.value) {
          canMove.value = false
          if (axisValue > 0.5 || dpadRight) {
            updateSelection(selectedIndex.value + 1)
          } else if (axisValue < -0.5 || dpadLeft) {
            updateSelection(selectedIndex.value - 1)
          }
          setTimeout(() => { canMove.value = true }, 200)
        }

        // Handle A button
        if (aButton && !lastAButtonState.value && !isTransitioning.value) {
          handleItemClick(selectedIndex.value)
          lastAButtonState.value = true
        } else if (!aButton) {
          lastAButtonState.value = false
        }
      }
    }
    
    animationFrameId = requestAnimationFrame(checkGamepadInternal)
  }

  checkGamepadInternal()
  return () => {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId)
    }
  }
}

onMounted(() => {
  window.addEventListener('gamepadconnected', gamepadConnectHandler)
  window.addEventListener('gamepaddisconnected', gamepadDisconnectHandler)

  const gamepadCleanup = checkGamepad()

  onUnmounted(() => {
    window.removeEventListener('gamepadconnected', gamepadConnectHandler)
    window.removeEventListener('gamepaddisconnected', gamepadDisconnectHandler)
    gamepadCleanup()
  })
})
</script>

<style scoped>
  .eclipse-menu {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: #0a0a0a;
    display: flex;
    justify-content: center;
    align-items: center;
    font-family: 'Orbitron', sans-serif;
    overflow: hidden;
  }
  
  .eclipse-wrapper {
    position: relative;
    width: 1920;
    height: 1080;
    display: flex;
    justify-content: center;
    align-items: center;
    transform: translateY(-40px);
  }
  
  .central-eclipse {
    position: relative;
    width: 250px;
    height: 250px;
    background: transparent;
    border-radius: 50%;
    overflow: hidden;
    box-shadow: 
      0 0 50px #ffffffb6,
      inset 0 0 70px #0e0e0e;
    border: 3px solid #ffffff;
    transition: all 0.3s ease;
  }
  
  .sun {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #000000;
    opacity: 1;
    border-radius: 50%;
    transform: scale(0.9);
  }
  
  .moon {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #000000;
    opacity: 1;
    border-radius: 50%;
    transform: scale(0.85);
  }
  
  .title {
    position: absolute;
    width: 100%;
    text-align: center;
    color: white;
    font-size: 2.5em;
    font-weight: 600;
    letter-spacing: 3px;
    text-shadow: 
      0 0 12px rgba(31, 152, 203, 0.6),
      0 0 18px rgba(0, 0, 0, 0.4);
    z-index: 10;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    opacity: 1;
    transition: all 0.3s ease;
  }
  

  
  .central-eclipse:hover .title {
    text-shadow: 
      0 0 15px rgba(255, 255, 255, 0.8),
      0 0 22px rgba(0, 195, 255, 0.6);
  }
  
  .menu-ring {
    position: absolute;
    width: 100%;
    height: 100%;
  }
  
  .menu-option {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 60px;
    height: 60px;
    margin: -30px;
    transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
  
  .option-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
  }
  
  .mini-eclipse {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background: linear-gradient(
      135deg, 
      rgba(255, 255, 255, 0.8) 0%, 
      rgba(255, 255, 255, 0.6) 100%
    );
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 
      0 0 20px rgba(255, 255, 255, 0.5),
      0 0 15px rgba(0, 195, 255, 0.2);
    border: 1px solid rgba(255, 255, 255, 0.5);
    transition: all 0.3s ease;
  }
  
  .mini-eclipse-inner {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    background: radial-gradient(
      circle at 30% 30%, 
      #000000 0%, 
      #1a1a1a 100%
    );
    box-shadow: 
      inset 0 0 10px rgba(0, 0, 0, 0.5),
      0 0 8px rgba(0, 0, 0, 0.3);
  }
  
  .option-text {
    margin-top: 8px;
    color: white;
    font-size: 0.7em;
    font-weight: 600;
    letter-spacing: 1px;
    text-shadow: 
      0 0 4px rgba(0, 0, 0, 0.5);
    transition: all 0.3s ease;
  }
  
  .menu-option.is-focused .mini-eclipse,
  .menu-option:hover .mini-eclipse {
    background: linear-gradient(
      135deg, 
      rgba(0, 195, 255, 0.8) 0%, 
      rgba(0, 195, 255, 0.6) 100%
    );
    box-shadow: 
      0 0 25px rgba(0, 195, 255, 0.6),
      0 0 20px rgba(255, 255, 255, 0.4);
    border-color: rgba(255, 255, 255, 0.7);
  }
  
  .menu-option.is-focused .mini-eclipse-inner,
  .menu-option:hover .mini-eclipse-inner {
    background: radial-gradient(
      circle at 30% 30%, 
      #ffffff 0%, 
      #e6e6e6 100%
    );
    box-shadow: 
      inset 0 0 10px rgba(0, 195, 255, 0.5),
      0 0 15px rgba(255, 255, 255, 0.6);
  }
  
  .menu-option.is-focused .option-text,
  .menu-option:hover .option-text {
    color: #00c3ff;
    text-shadow: 
      0 0 6px #00c3ff,
      0 0 10px rgba(255, 255, 255, 0.6);
  }
  
  .back-button {
    width: 75px;
    height: 50px;
    border-radius: 25px;
    background: #dfdfdf;
    box-shadow: 
      0 0 10px #FFFFFF;
    transition: all 0.3s ease;
  }
  
  .menu-option.is-focused .back-button,
  .menu-option:hover .back-button {
    background: #FFFFFF;
    box-shadow: 
      0 0 15px #000000;
  }
</style>