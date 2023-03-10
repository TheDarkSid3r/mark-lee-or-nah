<script lang="ts">
import { defineComponent } from "vue";
import ChoiceButton from "./components/ChoiceButton.vue";
import { marklee } from "./marklee";

const sizes = [[240, 360], [240, 160], [281, 360], [246, 360], [252, 360], [236, 360], [268, 360], [294, 360], [238, 240], [240, 234], [240, 235], [240, 240], [235, 360], [239, 360], [257, 360], [241, 360], [291, 360], [240, 680], [288, 360], [253, 360], [233, 360], [231, 360], [248, 360], [960, 720], [640, 960], [639, 960], [720, 960], [960, 640], [1366, 1372], [1344, 2015], [1365, 1356], [1343, 2015], [1440, 960], [1345, 2015], [641, 960], [1333, 1344], [960, 641]];

const minLoadingTime = 1000;
const sequence = ["up", "up", "down", "down", "left", "right", "left", "right", "b", "a", "start"];

export default defineComponent({
    data(): {
        isMarkLee: boolean;
        isLoading: boolean;
        loadingStart: number;
        image: HTMLImageElement;
        hasResult: boolean;
        wasCorrect: boolean;
        score: number;
        highScore: number;
        autoplay: boolean;
        sequenceNum: number;
    } {
        const highScore = localStorage.getItem("mlhighscore");
        return {
            isMarkLee: false,
            isLoading: true,
            loadingStart: -1,
            image: new Image(),
            hasResult: false,
            wasCorrect: false,
            score: 0,
            highScore: !highScore || isNaN(highScore as any) ? 0 : parseInt(highScore!),
            autoplay: false,
            sequenceNum: 0
        };
    },
    methods: {
        setup() {
            this.isMarkLee = Math.random() < 0.25;
            if (this.isMarkLee)
                this.load(marklee[Math.floor(Math.random() * marklee.length)]);
            else {
                const size = sizes[Math.floor(Math.random() * sizes.length)];
                this.load(`https://picsum.photos/seed/${Date.now()}/${size[0]}/${size[1]}`);
            }
        },
        load(src: string) {
            this.loadingStart = Date.now();
            this.isLoading = true;
            this.image.src = src;
        },
        loaded() {
            window.setTimeout(() => {
                this.isLoading = false;
                this.drawImage();
                if (this.autoplay) window.setTimeout(this.answerCorrectly, 150);
            }, Math.max(0, minLoadingTime - (Date.now() - this.loadingStart)));
        },
        drawImage() {
            const ctx = this.context;
            const canvas = ctx.canvas;
            const maxWidth = 350, maxHeight = 350;
            const image = this.image;
            const { width: imageWidth, height: imageHeight } = image;
            const scale = Math.min(maxWidth / imageWidth, maxHeight / imageHeight), newWidth = imageWidth * scale, newHeight = imageHeight * scale;
            canvas.width = newWidth;
            canvas.height = newHeight;
            ctx.clearRect(0, 0, newWidth, newHeight);
            ctx.drawImage(this.image, 0, 0, newWidth, newHeight);
        },
        choose(markLee: boolean) {
            if (this.autoplay) return;
            this.answer(markLee);
        },
        answer(markLee: boolean) {
            if (this.isLoading) return;
            this.wasCorrect = markLee === this.isMarkLee;
            if (this.wasCorrect) this.score++;
            else this.score = 0;
            if (this.score > this.highScore) {
                this.highScore = this.score;
                localStorage.setItem("mlhighscore", this.highScore.toString());
            }
            this.hasResult = true;
            this.setup();
        },
        answerCorrectly() {
            this.answer(this.isMarkLee);
        },
        startAutoplay() {
            this.autoplay = true;
            this.answerCorrectly();
        },
        keypress(e: KeyboardEvent) {
            if (this.autoplay) return;

            const key = this.getSequenceKey(e.key);
            if (!key) return;
            if (key === sequence[this.sequenceNum]) {
                this.sequenceNum++;
                if (this.sequenceNum >= sequence.length) this.startAutoplay();
            } else this.sequenceNum = 0;
        },
        getSequenceKey(key: string) {
            switch (key) {
                case "ArrowUp": return "up";
                case "ArrowDown": return "down";
                case "ArrowLeft": return "left";
                case "ArrowRight": return "right";
                case "a":
                case "A": return "a";
                case "b":
                case "B": return "b";
                case "Enter":
                case " ": return "start";
            }
        }
    },
    computed: {
        context() {
            return (this.$refs.canvas as HTMLCanvasElement).getContext("2d")!;
        }
    },
    mounted() {
        this.setup();
        this.image.addEventListener("load", this.loaded);
        window.addEventListener("keyup", this.keypress);
    },
    beforeUnmount() {
        window.removeEventListener("keyup", this.keypress);
    },
    components: { ChoiceButton }
});
</script>

<template>
    <div class="container">
        <div class="header">
            <div class="score">
                <div class="label">SCORE</div>
                <div class="number">{{ score }}</div>
            </div>
            <div class="score">
                <div class="label">HIGH SCORE</div>
                <div class="number">{{ highScore }}</div>
            </div>
        </div>
        <div class="canvas-container" :class="{ loading: isLoading }">
            <canvas ref="canvas" :width="350" :height="350"></canvas>
            <div :class="['result', hasResult ? wasCorrect ? 'correct' : 'incorrect' : 'none']">
                <div class="icon" v-if="hasResult"></div>
                <div class="text">{{ hasResult ? wasCorrect ? "CORRECT" : "INCORRECT" : "LOADING..." }}</div>
            </div>
        </div>
        <div class="choices" :class="{ disabled: isLoading }">
            <ChoiceButton @click="choose(true)">MARK LEE</ChoiceButton>
            <div class="or">OR</div>
            <ChoiceButton @click="choose(false)">NAH</ChoiceButton>
        </div>
    </div>
</template>

<style scoped lang="scss">
.container {
    padding: 30px;
    flex-grow: 1;
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    &>.header {
        display: flex;
        align-items: center;
        justify-content: center;

        &>.score {
            flex-shrink: 0;
            width: 120px;
            display: flex;
            flex-direction: column;
            color: #fff;
            text-align: center;

            &>.label {
                font-size: 16px;
                font-weight: 400;
            }

            &>.number {
                font-size: 20px;
                font-weight: 700;
                margin-top: 3px;
            }
        }
    }

    &>.canvas-container {
        margin: 30px;
        width: 350px;
        height: 350px;
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;

        &>canvas {
            position: relative;
            z-index: 1;
            transition: opacity 0.2s ease-out, transform 0.3s ease-out;
            opacity: 1;
            transform: none;
            border-radius: 15px;
        }

        &>.result {
            opacity: 0;
            width: 150px;
            height: 150px;
            position: absolute;
            z-index: 2;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0.8);
            pointer-events: none;
            border-radius: 15px;
            transition: opacity 0.2s ease-out, transform 0.2s ease-out;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;

            &>.icon {
                width: 50px;
                height: 50px;
            }

            &>.text {
                font-size: 16px;
                font-weight: 700;
                text-align: center;
                color: #000;
            }

            &.correct {
                background: rgba(59, 212, 59, 0.5);

                &>.icon {
                    background: url(./assets/correct.svg);
                }
            }

            &.incorrect {
                background: rgba(212, 59, 59, 0.5);

                &>.icon {
                    background: url(./assets/incorrect.svg);
                }
            }

            &.none {
                background: rgba(255, 255, 255, 0.5);
            }
        }

        &.loading {
            &>canvas {
                opacity: 0;
                transform: scale(0.9);
            }

            &>.result {
                opacity: 1;
                transform: translate(-50%, -50%);
                transition: transform 0.2s ease-out;
            }
        }
    }

    &>.choices {
        display: flex;
        align-items: center;
        justify-content: center;
        opacity: 1;
        transition: opacity 0.2s ease-out;

        &>button {
            flex-shrink: 0;
            border: none;
            outline: none;
            border-radius: 5px;
            padding: 10px 14px;
            font-size: 24px;
            font-weight: 700;
            cursor: pointer;
            background: #222;
            color: #fff;
            margin: 4px;
            text-align: center;
        }

        &.disabled {
            opacity: 0.2;

            &>button {
                cursor: default;
            }
        }

        &:not(.disabled)>button {
            &.hover {
                background: #111;
            }

            &.active {
                background: #fff;
                color: #000;
            }
        }

        &>.or {
            font-size: 18px;
            color: #fff;
            font-weight: 400;
            text-align: center;
            margin: 0 12px;
        }
    }
}
</style>