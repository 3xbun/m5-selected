<script setup>
import { subjects } from "./SelectedSubject.json";
import dayjs from 'dayjs';

import FlipCountdown from './components/FlipCountdown.vue';
import { computed, provide, ref } from "vue";

const targetTime = dayjs("2025-05-28 16:00")
const currentTime = dayjs()
provide("targetTime", targetTime)

const enableBtn = computed(() => targetTime.diff(currentTime) < 0 ? true : false)
</script>

<template>
    <FlipCountdown />
    <div class="container">
        <h1>วิชาเลือกเสรี</h1>
        <h2>มัธยมศึกษาปีที่ 5 - ปีการศึกษา 2568</h2>
        <div class="contents">
            <div class="subject-card" v-for="subject in subjects">
                <div class="img-wrapper">
                    <img :src="subject.image" :alt="subject.title">
                </div>
                <h4>
                    {{ subject.title }}
                </h4>
                <p class="condition">{{ subject.condition }}</p>
                <p>{{ subject.teacher }}</p>
                <p class="room"> {{ subject.room }}</p>
                <p class="desc">{{ subject.desc }}</p>
                <p v-if="subject.limit" class="limit">รับจำนวน <span>{{ subject.limit }}</span> คน</p>
                <a v-if="enableBtn" class="btn" :href="subject.link" target="_blank">ลงทะเบียน</a>
                <p v-else class="btn disable" >ลงทะเบียน</p>
            </div>
        </div>
    </div>
</template>

<style lang="css" scoped>
h1,
h2 {
    color: #fff;
}

.container {
    padding-top: 20vh;
    padding-bottom: 5vh;
    width: 90vw;
    max-width: 800px;
    text-align: center;
    margin: auto;
}

.contents {
    margin-top: 1em;
    display: flex;
    width: 100%;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 1em;
}

.subject-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    border-radius: 1em;
    width: 30%;
    background-color: #fff;
    padding: 1em;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1), 0 2px 8px rgba(0, 0, 0, 0.08);
    gap: .5em;
}

.img-wrapper {
    width: 100%;
    overflow: hidden;
    position: relative;
    /* Needed for absolute positioning of children */
    padding-bottom: 100%;
    /* This creates the square aspect ratio */
    height: 0;
    /* Collapses the height, letting padding define it */
}

img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    /* Ensures the image covers the square area */
}

.condition {
    color: #D3D3D3;
    background-color: #F0F0F0;
    width: fit-content;
    padding: 0 .5em;
    border-radius: 1em;
    margin: 0 auto;
}

.room {
    color: #fff;
    background-color: #4169E1;
    width: fit-content;
    padding: 0 .5em;
    border-radius: 1em;
}

.btn {
    /* Base Styling */
    padding: .5em 1em;
    /* Generous padding for a modern look */
    font-weight: 600;
    /* Semi-bold */
    color: #ffffff;
    /* White text */
    text-decoration: none;
    /* Remove underline if it's an <a> tag */
    border: none;
    /* No default border */
    border-radius: .5em;
    /* Rounded corners (12px) */
    cursor: pointer;
    width: max-content;

    margin-top: 1em;

    /* Cool Factor - Gradient Background */
    /* background-color: #4169E1; */
    background-image: linear-gradient(45deg, #4169E1 0%, #00008B 100%);
    /* Royal Blue to Dark Blue gradient */
    /* Cool Factor - Subtle Shadow for Depth */
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1), 0 2px 8px rgba(0, 0, 0, 0.08);

    margin-top: auto;
}

.disable {
    background-color: #D3D3D3;
    background-image: none;
    cursor: initial;
}

.limit span {
    font-weight: bold;
}

.desc {
    width: 100%;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    /* This is the key: limits to 3 lines */
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    /* Adds the "..." */
}

/* Media query for screens smaller than 768px (common tablet/mobile breakpoint) */
@media (max-width: 768px) {
    .subject-card {
        width: 45%;
        /* Two cards per row on medium screens */
        margin-bottom: 1em;
        /* Add some space between rows */
    }
}

/* Media query for screens smaller than 480px (common mobile breakpoint) */
@media (max-width: 480px) {
    .subject-card {
        width: 90%;
        /* Full width on small mobile screens */
        margin: auto;
        margin-bottom: 1em;
        /* Add some space between cards */
    }
}
</style>