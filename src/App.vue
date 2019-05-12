<template>
    <div id="app">
        <h1>{{ place.name }}</h1>
        <div v-if="noFoodOrGas" class="lost">
            <span v-if="noFood">No food left, you will starve.</span>
            <span v-else-if="noGas">No fuel left, you will run out of food.</span>
            <button class="lost-rerun" @click="rerun">rerun</button>
        </div>
        <div v-else-if="progress === 100" class="lost">
            <span>You die of age<br>with {{ blue }} Blue & {{ red }} Red</span>
            <button class="lost-rerun" @click="rerun">rerun</button>
        </div>
        <template v-else>
            <div class="choices">
                <div
                    v-for="choice in choices"
                    :class="['choice', {'choice--cannotAfford': cannotAfford(choice),'choice--canAfford': !cannotAfford(choice)}]"
                    @click="choose(choice)">
                    <div class="choice-to">
                        {{ choice.toValue }} {{ choice.toName }}
                    </div>
                    <div class="choice-arrow">↑</div>
                    <div class="choice-from">
                        {{ choice.fromValue }} {{ choice.fromName }}
                    </div>
                </div>
            </div>
            <div class="choices-pass" @click="pass">
                Pass
            </div>
        </template>
        <div class="cargo-divider"></div>
        <div class="cargo">
            <div v-for="item in cargo" :class="['cargo-item', {'cargo-item--low': item.amount === 1}]">
                {{ item.name }} {{ item.amount }}
            </div>
        </div>
        <div class="progress-wrapper">
            <div class="progress-bar" :style="{width: `${this.progress}%`}"></div>
        </div>
    </div>
</template>
<script>
    import gradients from './gradients.js';

    const Speed = 10;

    export default {
        name: 'app',
        data() {
            return {
                progress: 0,
                place: {
                    name: 'A1'
                },
                choices: [
                    { fromValue: 5, fromName: 'Blue', toValue: 2, toName: 'Red' },
                    { fromValue: 1, fromName: 'Blue', toValue: 3, toName: 'Food' },
                    { fromValue: 5, fromName: 'Blue', toValue: 2, toName: 'Gas' }
                ],
                cargo: [
                    { name: 'Blue', amount: 4 },
                    { name: 'Red', amount: 4 },
                    { name: 'Food', amount: 1 },
                    { name: 'Gas', amount: 2 },
                ]
            };
        },
        computed: {
            noFood() {
                return this.cargo.find(i => i.name === 'Food').amount === 0;
            },
            noGas() {
                return this.cargo.find(i => i.name === 'Gas').amount === 0;
            },
            noFoodOrGas() {
                return this.noFood || this.noGas;
            },
            red() {
                return this.cargo.find(i => i.name === 'Red').amount;
            },
            blue() {
                return this.cargo.find(i => i.name === 'Blue').amount;
            }
        },
        methods: {
            cannotAfford(choice) {
                return !this.canAfford(choice);
            },
            canAfford(choice) {
                return this.cargo.find(i => i.name === choice.fromName).amount >= choice.fromValue;
            },
            choose(choice) {
                if (this.canAfford(choice)) {
                    this.cargo.find(i => i.name === choice.fromName).amount -= choice.fromValue;
                    this.cargo.find(i => i.name === choice.toName).amount += choice.toValue;

                    this.nextPlace();
                }
            },
            pass() {
                this.nextPlace();
            },
            nextPlace() {
                this.progress += Speed;
                const currentSpice = this.choices[0].fromName;
                const nextSpice = currentSpice === 'Blue' ? 'Red' : 'Blue';
                this.place = {
                    name: randomItem(['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K'])
                        + randomItem(['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14'])
                };
                this.choices = [
                    {
                        fromValue: randomItem([1, 2, 3, 4, 5, 6, 7, 8]),
                        fromName: nextSpice,
                        toValue: randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: currentSpice
                    },
                    {
                        fromValue: randomItem([1, 2, 3, 4, 5, 6, 7, 8]),
                        fromName: nextSpice,
                        toValue: randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: 'Food'
                    },
                    {
                        fromValue: randomItem([1, 2, 3, 4, 5, 6, 7, 8]),
                        fromName: nextSpice,
                        toValue: randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: 'Gas'
                    }
                ];

                this.applyRandomGradient();

                this.cargo.find(i => i.name === 'Food').amount -= 1;
                this.cargo.find(i => i.name === 'Gas').amount -= 1;
            },
            applyRandomGradient() {
                document.querySelector('html').style = randomItem(gradients);
            },
            rerun() {
                window.location.reload();
            }
        },
        mounted() {
            this.applyRandomGradient();
        },
        components: {}
    }

    function randomItem(collection) {
        return collection[Math.round(Math.random() * (collection.length - 1))];
    }
</script>

<style lang="scss">
    html {
        background: #111;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: white;
        margin-top: 60px;
    }

    .lost {
        height: 220px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        font-size: 42px;
        font-weight: bold;
        user-select: none;
    }

    .lost-rerun {
        width: 200px;
        height: 42px;
        font-size: 18px;
        font-weight: bold;
        border: 1px solid rgba(255, 255, 255, .8);
        background: rgba(255, 255, 255, .12);
        color: rgba(255, 255, 255, .8);
        display: block;
        margin: 20px auto;

        &:hover {
            border: 2px solid rgba(255, 255, 255, 1);
            background: rgba(0, 0, 0, .1);
            color: rgba(255, 255, 255, 1);
        }
    }

    .choices {
        width: 800px;
        height: 220px;
        margin: 0 auto;
        display: flex;
        justify-content: space-evenly;
        align-items: center;
    }

    .choices-pass {
        display: inline-block;
        padding: 0 10px;
        color: rgba(255, 255, 255, .9);
        font-size: 32px;
        cursor: pointer;
        text-decoration: underline;
        user-select: none;
    }

    .choice {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 150px;
        height: 150px;
        border: 2px solid rgba(0, 0, 0, 0.2);
        outline: 1px solid rgba(0, 0, 0, 0.4);
        background: rgba(255, 255, 255, .12);
        font-size: 28px;
        font-weight: bold;
        user-select: none;
        box-sizing: border-box;
        color: rgba(255, 255, 255, .9);
    }

    .choice--canAfford:hover {
        background: rgba(255, 255, 255, .3);
        border: 2px solid #EEE;
        outline: 1px solid #AAA;
        cursor: pointer;
        color: white;

        .choice-arrow {
            font-size: 101%;
            //top: -18px;
        }
    }

    .choice--cannotAfford {
        color: rgba(255, 255, 255, .3);
        background: transparent;
    }

    .choice-from {
        font-size: 80%;
        /*color: rgba(255, 255, 255, .8);*/
    }

    .choice-arrow {
        display: inline-block;
        text-align: center;
        height: 20px;
        margin: 5px auto;
        position: relative;
        top: -15px;
    }

    .cargo {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 800px;
        margin: 0 auto;
        background: rgba(255, 255, 255, .05);
        border: 2px solid rgba(0, 0, 0, 0.2);
        outline: 1px solid rgba(0, 0, 0, 0.4);
        color: rgba(255, 255, 255, .9);
        user-select: none;

        &:hover {
        }
    }

    .cargo-divider {
        visibility: hidden;

        width: 600px;
        height: 2px;
        background: black;
        margin: 100px auto 25px auto;
    }

    .cargo-item {
        font-size: 24px;
        width: 125px;
        margin: 0 20px;
    }

    .cargo-item--low {
        color: rgba(255, 122, 122, 0.8);
        text-shadow: 0 0 2px rgba(255, 0, 0, 0.9);
    }

    .progress-wrapper {
        width: 800px;
        height: 20px;
        border: 2px solid rgba(255, 255, 255, .1);
        outline: 1px solid rgba(255, 255, 255, .2);
        margin: 12px auto;
    }

    .progress-bar {
        height: 100%;
        background: white;
    }
</style>
