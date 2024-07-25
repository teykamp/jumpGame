<template>
  <div style="display: flex; justify-content: center;">
    <canvas ref="gameCanvas" :width="width" :height="height"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

const { width, height } = {
  width: 800,
  height: 900
}


const scale = Math.max(Math.min(width / 1920, height / 1080), 0.75)

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
  x: width / 2 + 110,
  y: height,
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

const screenYOffset = computed(() => height - player.value.y)


const platforms: Platform[] = [
  {
    x: 0,
    y: height / scale - 5,
    width: width / scale,
    height: 25,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 0)'
      ctx.fillRect(platforms[0].x * scale, platforms[0].y * scale, platforms[0].width * scale, platforms[0].height * scale)
    }
  },
  {
    x: -25,
    y: 0,
    width: 25,
    height: height / scale,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 0)'
      ctx.fillRect(platforms[1].x * scale, platforms[1].y * scale, platforms[1].width * scale, platforms[1].height * scale)
    }
  },
  {
    x: width / scale,
    y: 0,
    width: 25,
    height: height / scale,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 0)'
      ctx.fillRect(platforms[2].x * scale, platforms[2].y * scale, platforms[2].width * scale, platforms[2].height * scale)
    }
  },
  {
    // bottom
    x: 0,
    y: -250,
    width: width / scale,
    height: 250,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'rgb(0, 0, 0, 0)'
      ctx.fillRect(platforms[3].x * scale, platforms[3].y * scale, platforms[3].width * scale, platforms[3].height * scale)
    }
  },
  {
    x: 0,
    y: height + 70 + screenYOffset.value,
    width: 400,
    height: 300,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[4].x * scale, platforms[4].y * scale, platforms[4].width * scale, platforms[4].height * scale)
    }
  },
  {
    x: width - 200 + 70,
    y: height + 70 + screenYOffset.value,
    width: 400,
    height: 300,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[5].x * scale, platforms[5].y * scale, platforms[5].width * scale, platforms[5].height * scale)
    }
  },
  {
    x: 200,
    y: height + 250 + screenYOffset.value,
    width: 600,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[6].x * scale, platforms[6].y * scale, platforms[6].width * scale, platforms[6].height * scale)
    }
  },
  {
    x: 300,
    y: 800 + screenYOffset.value,
    width: 100,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[7].x * scale, platforms[7].y * scale, platforms[7].width * scale, platforms[7].height * scale)
    }
  },
  {
    x: 500,
    y: 650 + screenYOffset.value,
    width: 100,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[8].x * scale, platforms[8].y * scale, platforms[8].width * scale, platforms[8].height * scale)
    }
  },
  {
    x: 700,
    y: 500 + screenYOffset.value,
    width: 500,
    height: 100,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[9].x * scale, platforms[9].y * scale, platforms[9].width * scale, platforms[9].height * scale)
    }
  },
  {
    x: 0,
    y: 200 + screenYOffset.value,
    width: 100,
    height: 900,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[10].x * scale, platforms[10].y * scale, platforms[10].width * scale, platforms[10].height * scale)
    }
  },
  {
    x: 450,
    y: 400 + screenYOffset.value,
    width: 100,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[11].x * scale, platforms[11].y * scale, platforms[11].width * scale, platforms[11].height * scale)
    }
  },
  {
    x: 200,
    y: 300 + screenYOffset.value,
    width: 100,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[12].x * scale, platforms[12].y * scale, platforms[12].width * scale, platforms[12].height * scale)
    }
  },
  {
    x: 200,
    y: 100 + screenYOffset.value,
    width: 1000,
    height: 50,
    draw: (ctx: CanvasRenderingContext2D) => {
      ctx.fillStyle = 'green'
      ctx.fillRect(platforms[13].x * scale, platforms[13].y * scale, platforms[13].width * scale, platforms[13].height * scale)
    }
  },

]

const gravity = 0.98
const jumpStrength = ref(20 * -1)
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
    const aCenterX = a.x + a.width / 2;
    const aCenterY = a.y + a.height / 2;
    const bCenterX = b.x + b.width / 2;
    const bCenterY = b.y + b.height / 2;

    const dx = aCenterX - bCenterX;
    const dy = aCenterY - bCenterY;
    const overlapX = (a.width / 2 + b.width / 2) - Math.abs(dx);
    const overlapY = (a.height / 2 + b.height / 2) - Math.abs(dy);

    if (overlapX < overlapY) {
      direction = dx > 0 ? 'left' : 'right';
    } else {
      direction = dy > 0 ? 'top' : 'bottom';
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
