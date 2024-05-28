<script>
export default {
    props: ['binary'],
    data() {
        return {
            digits: new Array()
        }
    },
    beforeMount() {
        for (let i = 1; i <= this.binary.length; i++) {
            let v = this.binary.charAt(i - 1) === '1'
            this.digits.push({
                id: i,
                value: v
            })
        }
    }
}
</script>

<template>
<div class="dip">
    <div class="switch" v-for="digit in digits">
        <div class="index">{{ digit.id }}</div>
        <div class="lever" :class="{on: digit.value}"><div class="dot"></div></div>
    </div>
</div>
</template>

<style>
.dip {
    display: flex;
    background-color: #8b6262;
    color: white;
    width: min-content;
    flex-direction: row;
    flex-wrap: nowrap;
    gap: 5px;
    padding: 8px;
    text-align: center;
    font-family: 'Roboto', sans-serif;
}

.lever {
    height: 50px;
    width: 15px;
    background-color: black;
    display: flex;
    flex-direction: column-reverse;
}

.lever.on {
    flex-direction: column;
}

.dot {
    background-color: white;
    height: 20px;
    width: 15px;
}
</style>