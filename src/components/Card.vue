<template>
    <div class="card">
        <div class="title">
            <div>{{cardValue}}</div>
            <div>{{cardSymbol}}</div>
        </div>
    </div>
</template>

<script>
export default {
    name: "CardComponent",
    props: {
        cardSymbol: String,
        cardValue: String
    },
    setup() {
    }
}
</script>

<style scoped>
.card {
    height: 160px;
    width: 110px;
    background-color: #fff;
    display: inline-block;
    border: 1px solid #eee;
}
.title {
    text-align: left;
    font-weight: 600;
    padding-left: 2px;
    padding-top: 2px;
}
</style>