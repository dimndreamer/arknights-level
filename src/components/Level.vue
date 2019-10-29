<template>
    <v-card>
        <v-card-text>
            <v-row>
                <v-col
                    cols="12"
                    sm="6"
                >
                    <v-row
                        class="star-group"
                        justify="space-between"
                        justify-sm="start"
                        style="margin: 0px;"
                    >
                        <v-btn
                            v-for="rarityItem in fields.rarityItems"
                            :key="rarityItem"
                            :style="rarityItem !== 1 ? 'margin-left: -10px;' : ''"
                            icon
                            :color="rarityItemColorMap[rarityItem - 1]"
                            @click="triggerRarityItem(rarityItem)"
                        >
                            <v-icon
                                large
                                style="transform: rotate(13.33deg);"
                            >
                                mdi-star
                            </v-icon>
                        </v-btn>
                    </v-row>
                </v-col>
                <v-col
                    class="current-evolve"
                    cols="6"
                    sm="auto"
                >
                    <v-row
                        justify="center"
                        justify-sm="end"
                    >
                        <!-- 当前精英化 -->
                        <v-btn
                            v-for="evolve in evolves"
                            :key="evolve.key"
                            icon
                            :color="evolve.key === currentEvolve ? 'yellow darken-2' : ''"
                            @click="triggerCurrentEvolve(evolve.key)"
                        >
                            <v-icon
                                large
                            >
                                {{ evolve.icon }}
                            </v-icon>
                        </v-btn>
                    </v-row>
                </v-col>
                <v-col
                    cols="6"
                    sm="auto"
                >
                    <v-row
                        justify="center"
                        justify-sm="start"
                    >
                        <!-- 目标精英化 -->
                        <v-btn
                            v-for="evolve in evolves"
                            :key="evolve.key"
                            icon
                            :disabled="evolve.key in disabledEvolves"
                            :color="evolve.key === targetEvolve ? 'yellow darken-2' : ''"
                            @click="triggerTargetEvolve(evolve.key)"
                        >
                            <v-icon
                                large
                            >
                                {{ evolve.icon }}
                            </v-icon>
                        </v-btn>
                    </v-row>
                </v-col>
            </v-row>
            <v-slider
                class="current-level mt-8"
                min="1"
                :max="maxCurrentLevel"
                :value="currentLevel"
                thumb-label="always"
                :color="options.alwaysMaxCurrentLevel ? 'yellow darken-3' : ''"
                :track-color="options.alwaysMaxCurrentLevel ? 'yellow darken-1' : ''"
                @change="onCurrentLevelChange"
            >
                <template v-slot:prepend>
                    <v-btn
                        :color="options.alwaysMaxCurrentLevel ? 'blue' : ''"
                        icon
                        @click="minimizeCurrentLevel"
                    >
                        <v-icon>
                            mdi-water
                        </v-icon>
                    </v-btn>
                </template>
                <template v-slot:append>
                    <v-btn
                        :color="options.alwaysMaxCurrentLevel ? 'yellow darken-3' : ''"
                        icon
                        @click="maximizeCurrentLevel"
                    >
                        <v-icon>
                            mdi-fire
                        </v-icon>
                    </v-btn>
                </template>
            </v-slider>
            <v-slider
                ref="targetLevelSlider"
                class="mt-4"
                min="1"
                :value="targetLevel"
                :max="maxTargetLevel"
                thumb-label="always"
                :rules="rules"
                @change="onTargetLevelChange"
            >
                <template v-slot:prepend>
                    <v-btn
                        icon
                        color="blue"
                        @click="minimizeTargetLevel"
                    >
                        <v-icon>
                            mdi-bomb-off
                        </v-icon>
                    </v-btn>
                </template>
                <template v-slot:append>
                    <v-btn
                        icon
                        color="red"
                        @click="maximizeTargetLevel"
                    >
                        <v-icon>
                            mdi-bomb
                        </v-icon>
                    </v-btn>
                </template>
            </v-slider>
        </v-card-text>
        <v-card-text class="result">
            <span class="orange--text">
                经验
            </span>
            <span class="orange--text text--lighten-3">
                {{ calc.exps.toLocaleString() }} 
            </span>
            <span class="blue--text">
                钱 
            </span>
            <v-tooltip top>
                <template v-slot:activator="{on}">
                    <span
                        class="blue--text text--lighten-3"
                        v-on="on"
                    >
                        {{ calc.costs.toLocaleString() }}
                    </span>
                </template>
                <span>
                    升级花费
                </span>
            </v-tooltip>
            <span style="font-size: 16px;">
                +
            </span>
            <v-tooltip top>
                <template v-slot:activator="{on}">
                    <span
                        class="blue--text text--lighten-3"
                        v-on="on"
                    >
                        {{ calc.evolveCosts.toLocaleString() }}
                    </span>
                </template>
                <span>
                    精英化花费
                </span>
            </v-tooltip>
            
            <!-- <div>经验：{{ calc.exps }}</div>
            <div>钱：{{ calc.costs }}</div>
            <div>钱：{{ calc.evolveCosts }}</div>
            <div>maxTargetLevel {{ maxTargetLevel }}</div>
            <div>maxTargetEvolve {{ maxTargetEvolve }}</div>
            <div>fields.targetLevel {{ fields.targetLevel }}</div> -->
        </v-card-text>
        <guide
            v-if="renderGuide"
            :steps="steps"
            @destroyMe="renderGuide = false"
        />
    </v-card>
</template>

<script>
import levelData from '@/gamedata/level.json';
import Guide from './Guide';

export default {
    name: 'Level',
    components: {
        'guide': Guide
    },
    data: () => ({
        fields: {
            rarity: 5,
            rarityItems: [1, 2, 3, 4, 5, 6],
            evolves: [0, 1, 2],
            currentEvolve: 0,
            currentLevel: 1,
            currentExp: '',
            targetEvolve: 0,
            targetLevel: 1,
            targetExp: '',
        },
        options: {
            alwaysMaxCurrentLevel: false,
            lockTargetLevel: false,
        },
        steps: [
            {
                content: '选择星级',
                target: '.star-group',
            },
            {
                content: '选择精英化',
                target: '.current-evolve',
            },
            {
                content: '选择等级',
                target: '.current-level',
            }
        ],
        renderGuide: false,
    }),
    computed: {
        rules() {
            return [
                this.rule,        
            ]
        },
        rule() {
            if(this.IsEvolveSame && this.targetLevel < this.currentLevel) {
                return '目标等级应大于当前等级';
            }else {
                return false;
            }
        },
        calc() {
            let exps = 0;
            let costs = 0;
            let evolveCosts = 0;
                
            if(this.currentEvolve < this.targetEvolve) {
                for(let i = this.currentEvolve; i <= this.targetEvolve; i++) {
                    let maxLevel = levelData.maxLevel[this.rarity][i];
                    if(i === this.currentEvolve) {
                        // 第一段，需要注意初始等级
                        for(let j = this.currentLevel - 1; j < maxLevel - 1; j++) {
                            exps += levelData.characterExpMap[i][j];
                            costs += levelData.characterUpgradeCostMap[i][j];
                        }
                    } else if(i === this.targetEvolve) {
                        // 最后一段，需要注意目标等级
                        for(let j = 0; j < this.targetLevel - 1; j++) {
                            exps += levelData.characterExpMap[i][j];
                            costs += levelData.characterUpgradeCostMap[i][j];
                        }
                    }else {
                        // 中间段，直接加上就行
                        for(let j = 0; j < maxLevel - 1; j++) {
                            exps += levelData.characterExpMap[i][j];
                            costs += levelData.characterUpgradeCostMap[i][j];
                        }
                    }
                }
            } else {
                for(let j = this.currentLevel - 1; j < this.targetLevel - 1; j++) {
                    exps += levelData.characterExpMap[this.currentEvolve][j];
                    costs += levelData.characterUpgradeCostMap[this.currentEvolve][j];
                }
            }

            // 计算精英化的花费
            if(this.currentEvolve < this.targetEvolve) {
                if(this.currentEvolve + 1 < this.targetEvolve) {
                    evolveCosts += levelData.evolveGoldCost[this.rarity][0];
                    evolveCosts += levelData.evolveGoldCost[this.rarity][1];
                } else {
                    evolveCosts += levelData.evolveGoldCost[this.rarity][this.targetEvolve - 1];
                }			
            }

            let results = {
                exps,
                costs,
                evolveCosts,
            }
            
            this.save();

            return results;
        },
        rarity() {
            // 因为值是绑定在item-group上的，所以就直接是zero-base
            return this.fields.rarity;
        },
        maxCurrentLevel() {
            return levelData.maxLevel[this.rarity][this.currentEvolve];
        },
        maxTargetLevel() {
            return levelData.maxLevel[this.rarity][this.targetEvolve];
        },
        currentLevel() {
            if(this.fields.currentLevel > this.maxCurrentLevel) {
                return this.maxCurrentLevel;
            }
            return this.fields.currentLevel;
        },
        targetLevel() {
            if(this.fields.targetLevel > this.maxTargetLevel) {
                return this.maxTargetLevel;
            }
            return this.fields.targetLevel;
        },
        evolves() {
            let evolveData = [
                {
                    icon: 'mdi-circle-medium',
                    key: 0
                },
                {
                    icon: 'mdi-chevron-down',
                    key: 1
                },
                {
                    icon: 'mdi-chevron-double-down',
                    key: 2
                }
            ];
            let temp = [evolveData[0]];
            levelData.evolveGoldCost[this.fields.rarity].forEach((cost, index) => {
                if(cost !== -1) {
                    temp.push(evolveData[index + 1]);
                }
            });
            return temp;
        },
        disabledEvolves() {
            let temp = [];
            this.evolves.slice(0, this.currentEvolve).forEach(el => {
                temp.push(el.key);
            });
            return temp;
        },
        maxCurrentEvolve() {
            let baseEvolve = 0;
            levelData.evolveGoldCost[this.rarity].forEach(cost => {
                if(cost !== -1) {
                    baseEvolve++;
                }
            });
            return baseEvolve;
        },
        maxTargetEvolve() {
            return this.maxCurrentEvolve;
        },
        currentEvolve() {
            if(this.fields.currentEvolve < this.maxCurrentEvolve) {
                return this.fields.currentEvolve;
            }
            return this.maxCurrentEvolve;
        },
        targetEvolve() {
            if(this.fields.targetEvolve < this.maxCurrentEvolve) {
                return this.fields.targetEvolve;
            }
            return this.maxTargetEvolve;
        },
        IsEvolveSame() {
            return this.targetEvolve === this.currentEvolve;
        },
        rarityItemColorMap() {
            let rarity = this.rarity;
            let colors = [
                '#b9b9b9', '#252625', 'deep-purple lighten-3',
                'amber accent-1', '#feb235'
            ];
            let colorMap = [];
            for(let i = 0; i < 6; i++) {
                colorMap.push(colors[0]);
            }

            if(rarity in [0, 1, 2]) {
                for(let i = 0; i <= rarity; i++) {
                    colorMap[i] = colors[1];
                }
            } else {
                for(let i = 0; i <= rarity; i++) {
                    colorMap[i] = colors[rarity - 1];
                }
            }
            
            return colorMap;
        },

    },
    watch: {
        maxCurrentLevel() {
            if(this.options.alwaysMaxCurrentLevel) {
                this.fields.currentLevel = this.maxCurrentLevel;
            }
        },
        // 即便是在created赋值，也会触发watch的内容
        // 'fields.currentLevel'() {
        //     if(this.fields.currentEvolve === this.fields.targetEvolve) {
        //         this.syncTargetLevel();
        //     }
        // }
    },
    mounted() {
        if(!localStorage.getItem('guide_finished') || localStorage.getItem('guide_finished') === 'false') {
            this.renderGuide = true;
        }
    },
    created() {
        let savedData = localStorage.getItem('levelData');
        if(savedData) {
            let savedObject = JSON.parse(savedData);
            this.fields.rarity = savedObject.rarity;
            this.fields.currentEvolve = savedObject.currentEvolve;
            this.fields.currentLevel = savedObject.currentLevel;
            this.fields.targetEvolve = savedObject.targetEvolve;
            this.fields.targetLevel = savedObject.targetLevel;
        }
    },
    methods: {
        triggerRarityItem(value) {
            this.fields.rarity = value - 1;
        },
        triggerCurrentEvolve(value) {
            // 点击当前值，会自动确定目标值，因为目标值总是大于等于当前值
            this.fields.currentEvolve = value;
        },
        triggerTargetEvolve(value) {
            this.fields.targetEvolve = value;
        },
        minimizeCurrentLevel() {
            this.fields.currentLevel = 1;
            this.options.alwaysMaxCurrentLevel = false;
        },
        maximizeCurrentLevel() {
            // 点击最大值之后，修改选项，再通过watch锁定最大值
            // 这个锁定的操作其实更多的是一个炫技的成分
            this.fields.currentLevel = this.maxCurrentLevel;
            this.options.alwaysMaxCurrentLevel = true;
        },
        onCurrentLevelChange(value) {
            this.options.alwaysMaxCurrentLevel = false;
            this.fields.currentLevel = value;
            this.syncTargetLevel();
        },
        maximizeTargetLevel() {
            this.onTargetLevelChange(this.maxTargetLevel);
        },
        minimizeTargetLevel() {
            this.fields.targetLevel = 1;
        },
        onTargetLevelChange(value) {
            // 通过监听change事件，实现释放鼠标的时候改变值
            this.fields.targetLevel = value;
            this.options.lockTargetLevel = true;
        },
        syncTargetLevel() {
            // 等级没有手动调整过，并且在同一级
            if(!this.options.lockTargetLevel && this.IsEvolveSame) {
                this.fields.targetLevel = this.fields.currentLevel;
            }
        },
        save() {
            let newData = {
                rarity: this.rarity,
                currentEvolve: this.currentEvolve,
                currentLevel: this.currentLevel,
                targetEvolve: this.targetEvolve,
                targetLevel: this.targetLevel,
            }
            localStorage.setItem('levelData', JSON.stringify(newData));
        }
    }
}
</script>

<style>
.v-slider {
    margin-top: 7px;
}

.result span {
    margin-left: 6px;
    font-size: 24px;
}
</style>
