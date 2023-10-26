<script setup lang="ts">
import { ref, } from 'vue';
import Box from './components/Box.vue'
// Define types for position and dimensions

const imageCount = ref(20);
const enableOverlap = ref(false)
const imagePositions = ref(randomizeImagePosition())
const animate = ref(true)


type Position = Placement & Size

interface Placement {
  left: number;
  top: number;
}
interface Size {
  width: number;
  height: number;
}

// Function to check if two rectangles overlap
function isOverlap(rect1: Position, rect2: Position): boolean {
  if (rect1.left + rect1.width > 100 || rect1.top + rect1.height > 100) {
    return true
  }
  if (rect2.left + rect2.width > 100 || rect2.top + rect2.height > 100) {
    return true
  }
  return (
    (rect1.left < rect2.left + rect2.width &&
      rect1.left + rect1.width > rect2.left) &&
    (rect1.top < rect2.top + rect2.height &&
      rect1.top + rect1.height > rect2.top)
  );
}
// console.log(isOverlap({ "width": 22, "height": 22, "left": 4, "top": 67 }, { "width": 15, "height": 23, "left": 13, "top": 63 }))
// Function to generate a chain of non-overlapping positions for images
// function randomizeImagePosition(
// ): Position[] {
//   const containerHeight = 100
//   const containerWidth = 100
//   const maxImageWidth = enableOverlap.value ? 25 - (Math.ceil((imageCount.value / 10)) * 6) : 25
//   const minImageWidth = enableOverlap.value ? 20 - (Math.ceil((imageCount.value / 10)) * 6) : 20
//   const maxImageHeight = enableOverlap.value ? 35 - (Math.ceil((imageCount.value / 10)) * 6) : 45
//   const minImageHeight = enableOverlap.value ? 30 - (Math.ceil((imageCount.value / 10)) * 6) : 30
//   const positions: Position[] = [];

//   // Helper function to generate random dimensions within the specified range
//   function getRandomDimensions(): Position {
//     const width = Math.floor(Math.random() * (maxImageWidth - minImageWidth + 1)) + minImageWidth;
//     const height = Math.floor(Math.random() * (maxImageHeight - minImageHeight + 1)) + minImageHeight;
//     return { width, height, left: 0, top: 0 };
//   }

//   function getRandomCordinates(dimensions: Partial<Position>): Partial<Position> {
//     return {
//       left: Math.floor(Math.random() * (containerWidth - (dimensions?.width ?? 0))),
//       top: Math.floor(Math.random() * (containerHeight - (dimensions?.height ?? 0))),
//     }
//   }

//   for (let i = 0; i < imageCount.value; i++) {
//     let position: Position;
//     let dimensions: Position;
//     let attempts = 0;
//     let overlap = false

//     do {
//       dimensions = getRandomDimensions();
//       // Generate random dimensions for the current image
//       position = {
//         ...dimensions,
//         ...getRandomCordinates(dimensions)
//       };

//       // Check if the new image overlaps with any existing images
//       const newRect = { left: position.left, top: position.top, width: dimensions.width, height: dimensions.height };
//       overlap = enableOverlap.value ? positions.some((existingRect) => isOverlap(existingRect, newRect)) : false;

//       // Limit the number of attempts to avoid infinite loops
//       attempts++;
//     } while (overlap && attempts < 1000);

//     // If 100 attempts are reached, break to avoid an infinite loop
//     if (attempts === 1000) {
//       console.warn("Unable to position all images without overlap.");
//       break;
//     }

//     // Save the non-overlapping position and dimensions
//     positions.push(position);
//   }

//   return positions;
// }

function randomizeImagePosition(
): Position[] {
  const divideFactor = (Math.ceil((imageCount.value / 7)) * 4)
  console.log(divideFactor)
  const maxImageWidth = 25 - (divideFactor > 19 ? 19 : divideFactor)
  const minImageWidth = 20 - (divideFactor > 18 ? 18 : divideFactor)
  const maxImageHeight = 35 - (divideFactor > 29 ? 29 : divideFactor)
  const minImageHeight = 30 - (divideFactor > 28 ? 28 : divideFactor)
  const positions: Position[] = [];

  // Helper function to generate random dimensions within the specified range
  function getRandomDimensions(): Size {
    const width = Math.floor(Math.random() * (maxImageWidth - minImageWidth + 1)) + minImageWidth;
    const height = Math.floor(Math.random() * (maxImageHeight - minImageHeight + 1)) + minImageHeight;
    return { width, height };
  }

  function randomFactor(): number {
    const random = Math.random() * 100
    return (random % 15) * (random > 50 ? 1 : -1)
  }

  function getRandomPlacement(prevPosition: Position, width: number): Placement {
    // Define possible placement directions
    const placements = [
      // TOP
      { left: prevPosition.left + randomFactor(), top: prevPosition.top - prevPosition.height - 5 },
      // BOTTOM
      { left: prevPosition.left + randomFactor(), top: prevPosition.top + prevPosition.height + 5 },
      // LEFT
      { left: prevPosition.left - width - 5, top: prevPosition.top + randomFactor() },
      // RIGHT
      { left: prevPosition.left + prevPosition.width + 5, top: prevPosition.top + randomFactor() },
    ];

    // Randomly select a placement direction
    return placements[Math.floor(Math.random() * placements.length)];
  }

  function getPreviousPosition(index: number): Position {
    if (index === 0) {
      return { left: 45, top: 45, height: 0, width: 0 }
    }
    return positions[Math.ceil(Math.random() * 100) % positions.length]
  }


  for (let i = 0; i < imageCount.value; i++) {
    let position: Position = { left: 50, top: 50, height: 0, width: 0 };
    let dimensions: Size;
    let attempts = 0;
    let overlap = false
    let prevPosition: Position;
    let placement: Placement;
    do {
      prevPosition = getPreviousPosition(i)
      for (let i = 0; i < 1000; i++) {
        dimensions = getRandomDimensions()
        placement = getRandomPlacement(prevPosition, dimensions.width)
        if (placement.left < 0 || placement.left > 100 || placement.top < 0 || placement.top > 100) {
          continue
        }
        position = {
          ...dimensions,
          ...placement
        };
        overlap = positions.some((existingRect) => isOverlap(existingRect, position))
        if (!overlap) {
          break;
        }
      }
      // Limit the number of attempts to avoid infinite loops
      attempts++;
    } while (overlap && attempts < 100);

    // If 100 attempts are reached, break to avoid an infinite loop
    if (attempts === 1000) {
      console.warn("Unable to position all images without overlap.");
      break;
    }

    // Save the non-overlapping position and dimensions
    positions.push(position);
  }

  return positions;
}

function shuffle() {
  animate.value = true
  imagePositions.value = randomizeImagePosition()
  setTimeout(() => {
    animate.value = false
  }, 300)
}

function triggerAnimation() {
  animate.value = true
  setTimeout(() => {
    animate.value = false
  }, 300)
}

function increment() {
  imageCount.value += 1
  shuffle()
}
function decrement() {
  imageCount.value -= 1
  shuffle()
}
function toggleOverlap() {
  enableOverlap.value = !enableOverlap.value
  shuffle()
}
</script>

<template>
  <div>
    <div class="wrapper">
      <Box v-for="i, index in Array(imageCount)" :animate="animate" :key="index" :index="index" :style="{
        position: 'absolute',
        width: imagePositions[index].width + '%',
        height: imagePositions[index].height + '%',
        left: imagePositions[index].left + '%',
        top: imagePositions[index].top + '%',
        transform: enableOverlap ? 'scale(1.3)' : ''
      }">
        {{ JSON.stringify(imagePositions[index]) + `${i ?? ''}` }}
      </Box>
    </div>
    <span>
      <button @click="shuffle">
        shuffle (current: {{ JSON.stringify(imageCount) }})
      </button>
      <button @click="decrement">
        decrement
      </button>
      <button @click="increment">
        increment
      </button>
      <button @click="toggleOverlap">
        toggle overlap (current: {{ JSON.stringify(enableOverlap) }})
      </button>
      <button @click="triggerAnimation">
        trigger animation
      </button>

    </span>
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  padding: 20px;
  box-sizing: border-box;
  position: relative;
  width: 100%;
  max-width: 100%;
  min-width: 100%;
  overflow: hidden;
  height: 40vw;
}
</style>
