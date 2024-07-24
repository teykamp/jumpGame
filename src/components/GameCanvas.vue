<template>
 <div style="display: flex;">
   <canvas ref="gameCanvas" :width="width" :height="height"></canvas>
   {{ player }}
 </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const { width, height } = {
  width: window.innerWidth - 200,
  height: window.innerHeight - 20
}

const scale = Math.min(width / 800, height / 600) // Uniform scaling factor

type CollisionDirection = 'left' | 'right' | 'top' | 'bottom' | 'none'

type Object = {
  x: number,
  y: number,
  width: number,
  height: number,
  draw: (ctx: CanvasRenderingContext2D) => void
}

type Player = Object & {
  dx: number
  dy: number
  jumping: boolean
  jumpDirection: number
  onPlatform: boolean
}

type Platform = Object

const gameCanvas = ref<HTMLCanvasElement | null>(null)

const player = ref<Player>({
  x: 400,
  y: 300,
  width: 50,
  height: 50,
  dx: 0,
  dy: 0,
  jumping: false,
  jumpDirection: 0,
  onPlatform: false,
  draw: (ctx: CanvasRenderingContext2D) => {
    ctx.fillStyle = 'red'
    ctx.fillRect(player.value.x * scale, player.value.y * scale, player.value.width * scale, player.value.height * scale)
  }
})

const platforms: Platform[] = [
  { 
    x: 300, 
    y: 500, 
    width: 200, 
    height: 20, 
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[0].x * scale, platforms[0].y * scale, platforms[0].width * scale, platforms[0].height * scale)
    }
  },
  { 
    x: 100, 
    y: 400, 
    width: 200, 
    height: 200, 
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[1].x * scale, platforms[1].y * scale, platforms[1].width * scale, platforms[1].height * scale)
    }
  },
  { 
    x: 500, 
    y: 300, 
    width: 200, 
    height: 20, 
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[2].x * scale, platforms[2].y * scale, platforms[2].width * scale, platforms[2].height * scale)
    }
  },
  { 
    x: 0, 
    y: height - 5, 
    width: width, 
    height: 5 , 
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 1)'
      ctx.fillRect(platforms[3].x, platforms[3].y, platforms[3].width, platforms[3].height)
    }
  },
  { 
    x: 0, 
    y: 0, 
    width: 5, 
    height: height, 
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 1)'
      ctx.fillRect(platforms[4].x, platforms[4].y, platforms[4].width, platforms[4].height)
    }
  },
]

const gravity = 0.98 
const jumpStrength = ref(20 * -1 )
const moveSpeed = 5

const keyStates: Record<string, boolean> = {}

const handleKeyDown = (e: KeyboardEvent) => {
  keyStates[e.key] = true
  if (e.key === ' ' && player.value.onPlatform) player.value.dx = 0
}

const handleKeyUp = (e: KeyboardEvent) => {
  keyStates[e.key] = false

  if (e.key === ' ' && player.value.onPlatform) {
    
    if (keyStates['ArrowLeft']) {
      player.value.jumpDirection = -1
      player.value.dx = player.value.jumpDirection * moveSpeed

    } else if (keyStates['ArrowRight']) {
      player.value.jumpDirection = 1
      player.value.dx = player.value.jumpDirection * moveSpeed

    } else {
      player.value.jumpDirection = 0
      
    }
    player.value.dy = jumpStrength.value
    player.value.jumping = true
  }
}

const checkCollisions = <T extends Object, U extends Object>(a: T, b: U): { isColliding: boolean, direction: CollisionDirection } => {
  const isColliding = (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  )

  let direction: CollisionDirection = 'none'

  if (isColliding) {
    const aCenterX = a.x + a.width / 2
    const aCenterY = a.y + a.height / 2
    const bCenterX = b.x + b.width / 2
    const bCenterY = b.y + b.height / 2

    const dx = aCenterX - bCenterX
    const dy = aCenterY - bCenterY
    const overlapX = (a.width / 2 + b.width / 2) - Math.abs(dx)
    const overlapY = (a.height / 2 + b.height / 2) - Math.abs(dy)

    if (overlapX < overlapY) {
      direction = dx > 0 ? 'left' : 'right'
    } else {
      direction = dy > 0 ? 'top' : 'bottom'
    }
  }

  return { isColliding, direction }
}



const update = (ctx: CanvasRenderingContext2D) => {
  
  player.value.dy += gravity
  
  if (!player.value.jumping && !keyStates[' ']) {
    if (keyStates['ArrowLeft'] && player.value.onPlatform) {
      player.value.dx = -moveSpeed
    } else if (keyStates['ArrowRight'] && player.value.onPlatform) {
      player.value.dx = moveSpeed
    } else if (player.value.onPlatform) {
      player.value.dx = 0
    }
  }

  player.value.x += player.value.dx
  player.value.y += player.value.dy

  
  player.value.onPlatform = false
  platforms.forEach((platform) => {
  const { isColliding, direction } = checkCollisions(player.value, platform)

  if (isColliding) {
    switch (direction) {
      case 'left':
        player.value.x = Math.max(platform.x + platform.width, player.value.x)
        player.value.dx *= -1
        break
      case 'right':
        player.value.x = Math.min(platform.x - player.value.width, player.value.x)
        player.value.dx *= -1

        break
      case 'top':
        player.value.y = Math.max(platform.y + platform.height, player.value.y)
        player.value.dy = 0
        player.value.jumping = false
        break
      case 'bottom':
        player.value.y = platform.y - player.value.height
        player.value.dy = 0
        player.value.jumpDirection = 0
        player.value.jumping = false
        player.value.onPlatform = true
        break
    }
  }
})

  ctx.clearRect(0, 0, width, height)
  platforms.forEach((platform) => {
    platform.draw(ctx)
  })

  player.value.draw(ctx)

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
