<template>
    <div id="app">
        <div class="history">
            <span v-for="entry in history" class="history-entry">{{ entry }}</span>
        </div>
        <h1>{{ place.name }}</h1>
        <div v-if="noFoodOrGas" class="lost">
            <span v-if="noFood">You starve</span>
            <span v-else-if="noGas">You drift without fuel</span>
            <button class="lost-rerun" @click="rerun">rerun</button>
        </div>
        <div v-else-if="progress === 100" class="lost">
            <span>You leave <br>with {{ blue }} Blue & {{ red }} Red</span>
            <button class="lost-rerun" @click="rerun">rerun</button>
        </div>
        <template v-else>
            <div class="choices">
                <div
                    v-for="choice in choices"
                    :class="['choice', {'choice--cannotAfford': cannotAfford(choice),'choice--canAfford': !cannotAfford(choice)}]"
                    @click="choose(choice)">
                    <template v-if="choice.toAll">
                        <div v-for="to, index in choice.toAll">
                            {{ to.value }} {{ to.name }}{{ index < choice.toAll.length - 1 ? ',' : '' }}
                        </div>
                    </template>
                    <div v-else class="choice-to">
                        {{ choice.toValue }} {{ choice.toName }}
                    </div>
                    <template v-if="choice.fromValue > 0">
                        <div class="choice-arrow">↑</div>
                        <div class="choice-from">
                            {{ choice.fromValue }} {{ choice.fromName }}
                        </div>
                    </template>
                </div>
            </div>
            <div class="choices-pass" @click="pass">
                Move on
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

    const Speed = 0.5;

    export default {
        name: 'app',
        data() {
            const history = JSON.parse(localStorage.getItem('history') || '[]');

            return {
                history,
                progress: 0,
                place: this.randomPlace(),
                choices: [
                    {
                        fromValue: 0,
                        fromName: 'Blue',
                        toValue: 3,
                        toName: 'Red'
                    },
                    {
                        fromValue: 0,
                        fromName: 'Blue',
                        toValue: 3,
                        toName: 'Food'
                    },
                    {
                        fromValue: 0,
                        fromName: 'Blue',
                        toValue: 3,
                        toName: 'Gas'
                    }
                ],
                cargo: [
                    { name: 'Blue', amount: 3 },
                    { name: 'Red', amount: 3 },
                    { name: 'Food', amount: 3 },
                    { name: 'Gas', amount: 3 },
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
        watch: {
            noFoodOrGas() {
                if (this.noFoodOrGas) {
                    this.history.push(this.noFood ? 'No food' : 'No fuel');
                    this.storeHistory();
                }
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
                    for (const to of choice.toAll ? choice.toAll : [{ name: choice.toName, value: choice.toValue }]) {
                        this.cargo.find(i => i.name === to.name).amount += to.value;
                    }

                    this.pass();
                }
            },
            pass() {
                this.nextPlace();

                this.cargo.find(i => i.name === 'Food').amount -= 1;
                this.cargo.find(i => i.name === 'Gas').amount -= 1;
            },
            nextPlace() {
                this.progress += Speed;
                const currentSpice = this.choices[0].fromName;
                const nextSpice = currentSpice === 'Blue' ? 'Red' : 'Blue';
                this.place = this.randomPlace();
                this.choices = this.randomChoices(nextSpice, currentSpice);

                this.applyRandomGradient();
            },
            randomPlace() {
                return {
                    name: randomItem(['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K'])
                        + randomItem(['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14'])
                };
            },
            randomChoices(fromSpice, toSpice) {
                const spiceFromValue = this.red === 0 && this.blue === 0 && Math.random() < .1
                    ? 0
                    : randomItem([1, 2, 3, 4, 5, 6, 7]);

                return [
                    {
                        fromValue: spiceFromValue,
                        fromName: fromSpice,
                        toValue: spiceFromValue === 0 ? 1 : randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: toSpice
                    },
                    {
                        fromValue: fromSpice === 'Red'
                            ? randomItem([1, 2, 3, 4])
                            : randomItem([1, 2, 3, 4, 5, 6, 7, 8]),
                        fromName: fromSpice,
                        toValue: randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: 'Food'
                    },
                    {
                        fromValue: fromSpice === 'Blue'
                            ? randomItem([1, 2, 3, 4])
                            : randomItem([1, 2, 3, 4, 5, 6, 7, 8]),
                        fromName: fromSpice,
                        toValue: randomItem([2, 3, 4, 5, 6, 7, 8]),
                        toName: 'Gas'
                    }
                ];
            },
            applyRandomGradient() {
                document.querySelector('html').style = randomItem(gradients);
            },
            rerun() {
                window.location.reload();
            },
            resetHistory() {
                this.history = [];
                this.storeHistory();
            },
            storeHistory() {
                localStorage.setItem('history', JSON.stringify(this.history));
            }
        },
        mounted() {
            this.applyRandomGradient();

            window.addEventListener('keydown', event => {
                if (event.key === '®') this.resetHistory();
            });
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
        color: rgba(255, 255, 255, .25);
        font-size: 32px;
        cursor: pointer;
        text-decoration: underline;
        user-select: none;

        &:hover {
            color: rgba(255, 255, 255, .8);
        }
    }

    .choice {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 150px;
        height: 150px;
        /*border: 2px solid rgba(0, 0, 0, 0.4);*/
        /*outline: 1px solid rgba(0, 0, 0, 0.2);*/
        box-shadow: 0px 0px 10px 1px rgba(0, 0, 0, 0.08);
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
        /*background: rgba(255, 255, 255, .05);*/
        /*border: 2px solid rgba(0, 0, 0, 0.2);*/
        /*outline: 1px solid rgba(0, 0, 0, 0.4);*/
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

    .history {
        position: absolute;
        top: 10px;
        left: 10px;
        color: white;

        display: flex;
        flex-direction: column;
        align-items: flex-start;
        flex-wrap: wrap;

        height: 95vh;
    }

    .history-entry {
        font-size: 48px;
        padding-right: 10px;
        color: rgba(0, 0, 0, .019);
        z-index: -1;
    }
</style>
