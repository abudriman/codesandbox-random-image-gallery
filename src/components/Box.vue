<script setup lang="ts">

const props = defineProps({
    animate: Boolean,
    index: Number
})



</script>

<template>
    <div class="box" :class="{ animate: props.animate }">
        <img :src="`https://picsum.photos/id/${200 - (index ?? 0) - 20}/200/300`" alt="">
        <!-- <p>
            <slot></slot>
        </p> -->
    </div>
</template>

<style scoped>
.box {
    background: #FFF;
    box-sizing: border-box;
    position: absolute;
}

.animate {
    animation: pop .3s ease-out;
}

.box:hover {
    z-index: 2;
    background: black;
    color: white;
}

.box img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

@keyframes pop {
    0% {
        scale: 0;
    }

    100% {
        scale: 1;
    }
}

p {
    word-break: break-all;
}
</style>
