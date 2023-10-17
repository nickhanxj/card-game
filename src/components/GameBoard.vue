<template>
    <div id="card-game-board">
        <div id="user-bottom" class="card-container">
            <Card v-for="(card, index) in users[0]" :key="index" :cardValue="card.cardValue" :cardSymbol="card.cardSymbol"/>
            <span>user-1</span>
            <span class="winner" v-show="winner === 0">Winner!!</span>
        </div>
        <div id="user-left" class="card-container">
            <Card v-for="(card, index) in users[1]" :key="index" :cardValue="card.cardValue" :cardSymbol="card.cardSymbol"/>
            <span>user-2</span>
            <span class="winner" v-show="winner === 1">Winner!!</span>
        </div>
        <div id="user-top" class="card-container">
            <Card v-for="(card, index) in users[2]" :key="index" :cardValue="card.cardValue" :cardSymbol="card.cardSymbol"/>
            <span>user-3</span>
            <span class="winner" v-show="winner === 2">Winner!!</span>
        </div>
        <div id="user-right" class="card-container">
            <Card v-for="(card, index) in users[3]" :key="index" :cardValue="card.cardValue" :cardSymbol="card.cardSymbol"/>
            <span>user-4</span>
            <span class="winner" v-show="winner === 3">Winner!!</span>
        </div>
        <div class="card-holder">
            <Card :cardValue="topCard.cardValue" :cardSymbol="topCard.cardSymbol"/>
            <div>
                Remain: {{cards.size()}}
                <button @click="() => play()">play</button>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import Card from './Card.vue'

export default {
    name: "GameBoard",
    components: {
        Card
    },
    setup() {
        class Card {
            constructor(cardSymbol, symbolWeight, cardValue, valueWeight) {
                this.cardSymbol = cardSymbol;
                this.cardValue = cardValue;
                this.symbolWeight = symbolWeight;
                this.valueWeight = valueWeight;
            }
        }
        class Stack {
            constructor(length) {
                this._stack = Array(length);
                this._maxLength = length;
                this._stackTop = -1;
            }

            push(data){
                // 判断是否栈满
                if(this._stackTop === this._maxLength-1){
                    return false;
                }

                this._stack[++this._stackTop] = data
                return true
            }

            pop(){
                // 判断是否栈空
                if(this.isEmpty()){
                    return null;
                }

                // 可以不必清空出栈的元素，栈顶指针移动即可。下次入栈会自动覆盖
                return this._stack[this._stackTop--];
            }

            top(){
                // 判断是否栈空
                if(this.isEmpty()){
                    return null;
                }
                return this._stack[this._stackTop]; 
            }
            
            isEmpty(){
                return this._stackTop === -1;
            }

            size(){
                return this._stackTop + 1;
            }

            toString(){
                return `Stack(${this._stack.slice(0, this.stackTop+1).toString()})`
            }
        }

        const cardSymbol = [
            {"symbol": "@", "weight": 4},
            {"symbol": "#", "weight": 3},
            {"symbol": "^", "weight": 2},
            {"symbol": "*", "weight": 1}
        ]

        const cardValue = [
            {"value": "2", "weight": 2},
            {"value": "3", "weight": 3},
            {"value": "4", "weight": 4},
            {"value": "5", "weight": 5},
            {"value": "6", "weight": 6},
            {"value": "7", "weight": 7},
            {"value": "8", "weight": 8},
            {"value": "9", "weight": 9},
            {"value": "10", "weight": 10},
            {"value": "J", "weight": 11},
            {"value": "Q", "weight": 12},
            {"value": "K", "weight": 13},
            {"value": "A", "weight": 14}
        ]
        const stack = new Stack(cardSymbol.length * cardValue.length);
        const defaultCard = new Card("No Cards!", 0, "", 0);
        const topCard = ref(defaultCard);
        const initCards = ref([]);
        const cards = ref(stack);
        const users = ref([[],[],[],[]])
        const winner = ref(-1)
        let user_index = 0
        let dealInterval;

        // 初始化牌
        const init = () => {
            const cards = []
            for (let i = 0; i < cardSymbol.length; i++) {
                for (let j = 0; j < cardValue.length; j++) {
                    const symbol = cardSymbol[i];
                    const value = cardValue[j];
                    cards.push(new Card(symbol.symbol, symbol.weight, value.value, value.weight));
                }
            }
            initCards.value = cards;
        }

        // 发牌
        const deal = () => {
            const cardsStack = cards.value;
            if(!cardsStack.isEmpty()) {
                const card = cardsStack.pop();
                const stackTopCard = cardsStack.top();
                topCard.value = stackTopCard ? stackTopCard : defaultCard;
                users.value[user_index].push(card);
                user_index++;
                if (user_index % 4 === 0) {
                    user_index = 0;
                }
            } else {
                afterCardsDealComplete();
            }
        }

        // 发牌完成后的操作
        const afterCardsDealComplete = () => {
            clearInterval(dealInterval);
            sortUserCards();
            winner.value = findWinner();
        }

        // 开始游戏
        const play = () => {
            resetUsers();
            shuffle();
            dealInterval = setInterval(() => {
                deal();
            }, 50)
        }

        // 重置用户手牌
        const resetUsers = () => {
            users.value = [[],[],[],[]];
            winner.value = -1;
        }

        // 洗牌
        const shuffle = () => {
            const shuffledCards = doShuffle(initCards.value);
            shuffledCards.map(card => stack.push(card));
            topCard.value = stack.top();
        }

        // 洗牌算法
        const doShuffle = (arr) => {
            let result = arr;
            let j;
            for (let i = result.length - 1; i > 0; i--) {
                j = Math.floor(Math.random() * (i + 1));
                [result[i], result[j]] = [result[j], result[i]];
            }
            return result;
        }

        // 对用户手牌进行排序
        const sortUserCards = () => {
            const sortedCards = [];
            users.value.forEach(userCards => {
                doSort(userCards);
                sortedCards.push(userCards);
            })
            users.value = sortedCards;
        }

        // 排序：cardSymbol, symbolWeight, cardValue, valueWeight
        const doSort = (cards) => {
            cards.sort((c1, c2) => {
                const valueCompareResult = c1.valueWeight - c2.valueWeight;
                if (valueCompareResult !== 0) {
                    return valueCompareResult;
                }
                return c1.symbolWeight - c2.symbolWeight;
            })
        }

        // 找出赢家
        const findWinner = () => {
            const usersList = users.value;
            const userCountList = [];
            for (let i = 0; i < usersList.length; i++) {
                const result = findSameCards(i, usersList[i]);
                userCountList.push(result);
            }

            const maxCount = findMaxSameCountUsers(userCountList);
            const maxCountUsers = userCountList.filter(u => u['maxCount'] == maxCount);
            if (maxCountUsers.length == 1) { 
                // 同样牌最多者获胜
                return maxCountUsers[0]['userIndex'];
            }
            // 如果相同牌一样多，先比较牌面
            const maxValueWeight = findMaxCardValue(maxCountUsers);
            const maxCardValueUsers = userCountList.filter(u => u['maxCountCardValueWeight'] == maxValueWeight);
            if (maxCardValueUsers.length == 1) { 
                // 如果相同牌一样多，先比较牌面
                return maxCardValueUsers[0]['userIndex'];
            }
            // 如果相同牌一样多，且牌面一样多，则比较花色
            return findMaxSymbol(maxCardValueUsers);
        }

        const findMaxSymbol = (maxCardValueUsers) => {
            let maxSymbolWeight = 0;
            let maxSymbolUser = -1;
            maxCardValueUsers.forEach(user => {
                const sw = user['maxCountCardSymbolWeight'];
                if (sw > maxSymbolWeight) {
                    maxSymbolWeight = sw;
                    maxSymbolUser = user['userIndex'];
                }
            })
            return maxSymbolUser;
        }

        const findMaxCardValue = (maxCountUsers) => {
            let maxValueWeight = 0;
            maxCountUsers.forEach(user => {
                const vw = user['maxCountCards'][0]['valueWeight'];
                if (vw > maxValueWeight) {
                    maxValueWeight = vw;
                }
            })
            return maxValueWeight;
        }

        const findMaxSameCountUsers = (userCountList) => {
            const maxCount = userCountList.sort((u1, u2) => {
                return u2['maxCount'] - u1['maxCount']
            })[0]['maxCount'];
            return maxCount;
        }

        const findSameCards = (userIndex, cards) => {
            const countMap = {};
            cards.forEach(card => {
                const cardValue = card.cardValue;
                const count = countMap[cardValue];
                if (count) {
                    countMap[cardValue] = count + 1;
                } else {
                    countMap[cardValue] = 1;
                }
            })
            let maxCount = 0;
            let maxCoundCardValue;
            for (let s in countMap) {
                const cnt = countMap[s];
                if (cnt > maxCount) {
                    maxCount = cnt;
                    maxCoundCardValue = s;
                }
            }
            const maxCountCards = cards.filter(card => card.cardValue === maxCoundCardValue);
            const maxCountCardValueWeight = maxCountCards[0] ? maxCountCards[0]['valueWeight'] : -1;
            const maxCountCardSymbolWeight = maxCountCards.sort((m1, m2) => {
                return m2['symbolWeight'] - m1['symbolWeight'];
            })[0]['symbolWeight']
            return {
                "userIndex": userIndex,
                "maxCount": maxCount,
                "maxCountCards": maxCountCards,
                "maxCountCardValueWeight": maxCountCardValueWeight,
                "maxCountCardSymbolWeight": maxCountCardSymbolWeight,
            };
        }

        onMounted(() => {
            init();
            shuffle();
        });

        return {
            cards,
            topCard,
            users,
            winner,
            play
        }
    }
}
</script>

<style scoped>
#card-game-board {
    position: relative;
    width: 100vw;
    height: 100vh;
    background-color: aquamarine;
}
.card-holder {
    width: 150px;
    height: 200px;
    position: absolute;
    bottom:50%;
    margin-bottom: -50px;
    left: 50%;
    margin-left: -50px;
}
#user-top {
    top: 0;
    width: 100vw;
    text-align: center;
}

#user-bottom {
    width: 100vw;
    text-align: center;
    bottom: 0;
}

#user-left {
    bottom:50%;
    margin-bottom: -50px;
    left: 0;
}

#user-right {
    bottom:50%;
    margin-bottom: -50px;
    right: 0;
}
.card-container {
    position: absolute;
    padding: 5px;
}
.card-container > div:not(:first-child) {
    margin-left: -80px;
}

.card-container > span {
    display: block;
    text-align: center;
}
.winner {
    font-weight: 600;
    color: red;
}
</style>