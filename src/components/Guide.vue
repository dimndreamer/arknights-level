<template>
    <div class="guide">
        <div
            class="mask"
            :style="styles.maskStyle"
        />
        <v-card
            :key="currentStepIndex"
            class="step"
            :style="styles.stepStyle"
        >
            <v-card-title>
                {{ step.content }}
            </v-card-title>
            <v-card-actions>
                <v-btn
                    text
                    min-width="0"
                    color="red"
                    @click="triggerStop"
                >
                    跳过
                </v-btn>
                <v-btn
                    text
                    min-width="0"
                    color="green"
                    @click="triggerNext"
                >
                    知道了
                </v-btn>
            </v-card-actions>
        </v-card>
    </div>
</template>

<script>
export default {
    name: 'Guide',
    props: {
        steps: {
            type: Array,
            required: true
        },
    },
    data: () => ({
        currentStepIndex: 0,
        styles: {
            styleStyle: '',
            maskStyle: '',
        }
    }),
    computed: {
        isEnd() {
            return this.currentStepIndex === this.steps.length;
        },
        step() {
            return this.steps[this.currentStepIndex];
        },
    },
    created() {
        window.addEventListener('resize', this.main);
        this.main();
    },
    destroyed() {
        window.removeEventListener('resize', this.main);
    },
    methods: {
        main() {
            if(!this.isEnd) {
                let targetElement = document.querySelector(this.step.target);
                let rect = targetElement.getBoundingClientRect();

                let stepStyle = `
                    position: fixed;
                    top: ${rect.y + rect.height + 8}px;
                    left: ${rect.x}px;
                `;

                let maskStyle = `
                    position: absolute;
                    top: ${rect.y}px;
                    left: ${rect.x}px;
                    width: ${rect.width}px;
                    height: ${rect.height}px;
                `;

                this.styles = {
                    stepStyle,
                    maskStyle,
                }
            } else {
                this.$emit('destroyMe');
                localStorage.setItem('guide_finished', true);				
            }
        },
        triggerNext() {
            this.currentStepIndex++;
            this.main();
        },
        triggerStop() {
            this.currentStepIndex = this.steps.length;
            this.main();
        }
    }

}
</script>

<style scoped>
.guide {
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 10001;
}

.mask {
    box-shadow: 0 0 0 200vw rgb(0, 0, 0, 0.4);
}

@keyframes scaleIn {
    0% {
        transform: scale(0);
    }
    50% {
        transform: scale(1.1);
    }
    100% {
        transform: scale(1);
    }
}
.step {
    animation: scaleIn 0.25s ease-in-out 0s 1 forwards;
}
</style>
