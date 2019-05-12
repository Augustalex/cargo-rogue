<template>
    <div id="app">
        <h1>{{ place.name }}</h1>
        <div class="choices">
            <div v-for="choice in choices" class="choice">
                <div class="choice-to">
                    {{ choice.toValue }} {{ choice.toName }}
                </div>
                <div class="choice-arrow">↑</div>
                <div class="choice-from">
                    {{ choice.fromValue }} {{ choice.fromName }}
                </div>
            </div>
        </div>
        <div class="cargo-divider"></div>
        <div class="cargo">
            <div v-for="item in cargo" class="cargo-item">
                {{ item.name }} {{ item.amount }}
            </div>
        </div>
    </div>
</template>
<script>
    import gradients from './gradients.js';

    export default {
        name: 'app',
        data() {
            return {
                place: {
                    name: 'A1'
                },
                choices: [
                    { fromValue: 2, fromName: 'Blue', toValue: 1, toName: 'Red' },
                    { fromValue: 1, fromName: 'Blue', toValue: 1, toName: 'Food' },
                    { fromValue: 4, fromName: 'Blue', toValue: 1, toName: 'Gas' }
                ],
                cargo: [
                    { name: 'Blue', amount: 4 },
                    { name: 'Food', amount: 4 },
                    { name: 'Gas', amount: 4 },
                ]
            };
        },
        methods: {
            choose(choice) {

            }
        },

        mounted() {
            const items = gradients.length;
            const index = Math.round(Math.random() * (items - 1));
            document.querySelector('html').style = gradients[index];
        },
        components: {}
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
        color: #2c3e50;
        margin-top: 60px;
    }

    .choices {
        width: 800px;
        margin: 0 auto;
        display: flex;
        justify-content: space-evenly;
    }

    .choice {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 150px;
        height: 150px;
        border: 2px solid #00004d;
        outline: 1px solid #000033;
        background: #4d4d66;
        color: white;
        font-size: 28px;
        font-weight: bold;
        user-select: none;

        &:hover {
            background: #737399;
            border: 2px solid blue;
            outline: 1px solid purple;
            cursor: pointer;
            transform: scale(1.01);

            .choice-arrow {
                font-size: 130%;
                top: -20px;
            }
        }
    }

    .choice-from {
        font-size: 80%;
        color: rgba(255, 255, 255, .8);
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
        background: #e4e4f0;
        border: 1px solid black;
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
</style>
