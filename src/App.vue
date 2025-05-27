<script setup>
import { subjects } from "./SelectedSubject.json";
</script>

<template>
    <div class="container">
        <h1>วิชาเลือกเสรี</h1>
        <h2>มัธยมศึกษาปีที่ 5 - ปีการศึกษา 2568</h2>
        <div class="contents">
            <div class="subject-card" v-for="subject in subjects">
                <img :src="subject.image" :alt="subject.title">
                <h4>
                    {{ subject.title }}
                </h4>
                <p class="condition">{{ subject.condition }}</p>
                <p>{{ subject.teacher }} - {{ subject.room }}</p>
                <p>{{ subject.desc }}</p>
                <p class="limit">รับจำนวน {{ subject.limit }} คน</p>
                <a class="btn" :href="subject.link" target="_blank">ลงทะเบียน</a>
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
}

.subject-card {
    position: relative;
    border-radius: 1em;
    width: 30%;
    background-color: #fff;
    padding: 1em;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1), 0 2px 8px rgba(0, 0, 0, 0.08);

}

img {
    width: 100%;
    margin-bottom: 1em;
}

.condition {
    color: #D3D3D3;
    background-color: #F0F0F0;
    width: fit-content;
    padding: 0 .5em;
    border-radius: 1em;
    margin: auto;
}

.btn {
    position: absolute;
    bottom: 1em;
    left: 50%;
    transform: translateX(-50%);
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
    background-color: black;
    /* background-color: linear-gradient(45deg, #4169E1 0%, #00008B 100%); */
    /* Royal Blue to Dark Blue gradient */
    /* Cool Factor - Subtle Shadow for Depth */
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1), 0 2px 8px rgba(0, 0, 0, 0.08);
}

.limit {
    padding-bottom: 3em;
}
</style>