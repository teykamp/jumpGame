<template>
  <canvas ref="gameCanvas" width="800" height="600"></canvas>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Player {
  x: number
  y: number
  width: number
  height: number
  dx: number
  dy: number
  jumping: boolean
  jumpDirection: number
  onPlatform: boolean
}

interface Platform {
  x: number
  y: number
  width: number
  height: number
}

const gameCanvas = ref<HTMLCanvasElement | null>(null)
const player: Player = {
  x: 400,
  y: 300,
  width: 50,
  height: 50,
  dx: 0,
  dy: 0,
  jumping: false,
  jumpDirection: 0,
  onPlatform: false,
}

const platforms: Platform[] = [
  { x: 300, y: 500, width: 200, height: 20 },
  { x: 100, y: 400, width: 200, height: 20 },
  { x: 500, y: 300, width: 200, height: 20 },
]

const gravity = 0.5
const jumpStrength = -15
const moveSpeed = 5

const keyStates: Record<string, boolean> = {}

const handleKeyDown = (e: KeyboardEvent) => {
  keyStates[e.key] = true
}

const handleKeyUp = (e: KeyboardEvent) => {
  keyStates[e.key] = false

  if (e.key === ' ' && player.onPlatform) {
    
    if (keyStates['ArrowLeft']) {
      player.jumpDirection = -1
    } else if (keyStates['ArrowRight']) {
      player.jumpDirection = 1
    } else {
      player.jumpDirection = 0
    }
    player.dy = jumpStrength
    player.jumping = true
    player.onPlatform = false 
  }
}

const isColliding = (player: Player, platform: Platform) => {
  return (
    player.x < platform.x + platform.width &&
    player.x + player.width > platform.x &&
    player.y < platform.y + platform.height &&
    player.y + player.height > platform.y
  )
}

const update = (ctx: CanvasRenderingContext2D) => {
  
  player.dy += gravity

  
  if (!player.jumping && !keyStates[' ']) {
    if (keyStates['ArrowLeft']) {
      player.dx = -moveSpeed
    } else if (keyStates['ArrowRight']) {
      player.dx = moveSpeed
    } else {
      player.dx = 0
    }
  }

  if (player.jumping) {
    player.dx = player.jumpDirection * moveSpeed
  }

  player.x += player.dx
  player.y += player.dy

  
  player.onPlatform = false
  platforms.forEach((platform) => {
    if (isColliding(player, platform)) {
      // add bouncing off platform x here
      if (player.dy > 0) {
        player.y = platform.y - player.height
        player.dy = 0
        player.jumping = false
        player.onPlatform = true 
      } else if (player.dy < 0) {
        player.y = platform.y + platform.height
        player.dy = 0
      }
      if (!player.jumping) {
        player.dx = 0 
      }
    }
  })

  if (player.y + player.height > 600) {
    player.y = 600 - player.height
    player.dy = 0
    player.jumping = false
    player.onPlatform = true 
    player.dx = 0 
  }

  ctx.clearRect(0, 0, 800, 600)

  ctx.fillStyle = 'green'
  platforms.forEach((platform) => {
    ctx.fillRect(platform.x, platform.y, platform.width, platform.height)
  })

  ctx.fillStyle = 'red'
  ctx.fillRect(player.x, player.y, player.width, player.height)

  requestAnimationFrame(() => update(ctx))
}

onMounted(() => {
  const canvas = gameCanvas.value
  if (canvas) {
    const ctx = canvas.getContext('2d')
    if (ctx) {
      window.addEventListener('keydown', handleKeyDown)
      window.addEventListener('keyup', handleKeyUp)
      update(ctx)
    }
  }
})
</script>

<style scoped>
canvas {
  border: 1px solid #000;
}
</style>
