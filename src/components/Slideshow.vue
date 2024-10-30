<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from 'vue';

type Slide = {
    src: string;
    type: "image" | "video";
};

const slides = ref<Slide[]>([]);
const activeSlide = ref(0);
const cycle = ref(true);
const skipCycle = ref(false);
const video = ref<HTMLVideoElement | null>(null);

function shuffleArray(array: any[]) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

onMounted(async () => {
    try {
        const response = await fetch('/static-assets/slideshow/');
        const text = await response.text();

        const parser = new DOMParser();
        const doc = parser.parseFromString(text, 'text/html');
        const links = doc.querySelectorAll('a');

        slides.value = Array.from(links)
            .map(link => link.href)
            .filter(href => href.match(/\.(png|jpe?g|gif|webp|mp4)$/))
            .map(href => ({
                src: href.replace(/^(https?:\/\/[^\/]+)\//, '$1/static-assets/slideshow/'),
                type: href.endsWith('.mp4') ? 'video' : 'image'
            }));
    } catch (error) {
        console.error('Error loading images:', error);
    }

    shuffleArray(slides.value);
    console.log(slides.value);

    setInterval(() => {
        if (skipCycle.value) {
            skipCycle.value = false;
            return;
        }
        if (cycle.value) {
            activeSlide.value = (activeSlide.value + 1) % slides.value.length;
        }
    }, 5000);
});

watch(activeSlide, () => {
    if (slides.value[activeSlide.value].type === "video") {
        cycle.value = false;
        nextTick(() => {
            video.value = document.querySelector('video') as HTMLVideoElement;
            video.value.currentTime = 0;
            video.value.play();
            video.value.onended = () => {
                setTimeout(() => {
                    activeSlide.value = (activeSlide.value + 1) % slides.value.length;
                    skipCycle.value = true;
                    cycle.value = true;
                }, 2500);

            };
        });
    } else {
        if (video.value) {
            video.value.pause();
        }
        cycle.value = true;
    }

    if (activeSlide.value === slides.value.length - 1) {
        shuffleArray(slides.value);
    }
});

</script>

<template>
    <v-carousel v-model="activeSlide" :touch="{
        left: () => { activeSlide++; skipCycle = true; },
        right: () => { activeSlide--; skipCycle = true; }
    }" hide-delimiters show-arrows="hover">

        <template v-slot:prev="{ props }">
            <v-btn class="custom-arrow left" icon="$prev" @click="() => { props.onClick(); skipCycle = true; }"></v-btn>
        </template>

        <template v-slot:next="{ props }">
            <v-btn class="custom-arrow right" icon="$next"
                @click="() => { props.onClick(); skipCycle = true; }"></v-btn>
        </template>

        <v-carousel-item v-for="(slide, i) in slides" :key="i">
            <v-sheet class="fit-container">
                <v-img :src="slide.src" class="fit-image" v-if="slide.type === `image`" />
                <video :src="slide.src" class="fit-image" controls autoplay muted v-else-if="slide.type === `video`" />
            </v-sheet>
        </v-carousel-item>
    </v-carousel>
</template>

<style lang="scss" scoped>
.v-carousel {
    height: 100vh;
}

.v-carousel-item {
    display: flex;
    justify-content: center;
    align-items: center;
}

.fit-container {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.fit-image {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
}

.custom-arrow {
    display: none;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}

.custom-arrow.left {
    left: 10px;
}

.custom-arrow.right {
    right: 10px;
}

.v-carousel:hover .custom-arrow {
    display: block;
}
</style>