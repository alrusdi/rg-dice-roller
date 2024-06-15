<script setup lang="ts">
import { ref } from 'vue'

const {playersCount} = defineProps<{ playersCount: number }>();

interface Dice {
    id: string,
    value: string,
    color: string,
}

const BL_VALUES = ["bl1", "bl2", "bl3", "bl4", "bl5", "bl6"];
const GN_VALUES = ["gn1", "gn2", "gn3", "gn4", "gn5", "gn6"];
const OR_VALUES = ["or1", "or2", "or3", "or4", "or5", "or6"];
const PR_VALUES = ["pr1", "pr2", "pr3", "pr4", "pr5", "pr6"];

const dices: Dice[] = [
    {id: "bl_1", value: BL_VALUES[0], color: "blue"},
    {id: "bl_2", value: BL_VALUES[2], color: "blue"},

    {id: "gn_1", value: GN_VALUES[0], color: "green"},
    {id: "gn_2", value: GN_VALUES[2], color: "green"},

    {id: "or_1", value: OR_VALUES[0], color: "orange"},
    {id: "or_2", value: OR_VALUES[2], color: "orange"},

    {id: "pr_1", value: PR_VALUES[0], color: "purple"},
    {id: "pr_2", value: PR_VALUES[2], color: "purple"},
];

const initialSlots = {
    el: [null],

    p1: [null, null],
    p2: [null, null],
    p3: [null, null],
    p4: [null, null],

    bl: [dices[0], dices[1]],
    gn: [dices[2], dices[3]],
    or: [dices[4], dices[5]],
    pr: [dices[6], dices[7]],
};

const slots = ref(deepCopy(initialSlots));

var currentElement: any = null;

var isDragging = ref(false);

function deepCopy(val: any) {
    return JSON.parse(JSON.stringify(val));
}

function allowDrop(ev: any) {
    ev.preventDefault();
    return false;
}

function drag(ev: any) {
    const src = ev.target.parentElement;
    console.log(src);
    isDragging.value = true;
    currentElement = {
        draggableId: ev.target.id,
        sourceId: src.id
    };
}

function copyData(slot: Dice) {
    return {
        id: slot.id,
        value: slot.value,
        color: slot.color
    }
}

function setPlayerDice(p: Dice[], data: Dice) {
    if (!p[0]) {
        p[0] = data;
    } else {
        p[1] = data;
    }
}

function getData(slots: any, idx: number): Dice {
    var newData = copyData(slots[idx]);
    slots[idx] = null;
    return newData;
}

function isIndicesMatchingByColor(srcId: string, tgtId: string): boolean {
    return srcId.slice(4, 6) === tgtId.slice(0, 2)
}

function getSlotById(tgtId: string): Dice[] {
    var key = tgtId.slice(-3, -1);
    if (key.slice(-1) === "_") {
        key = tgtId.slice(-4, -2);
    }
    return slots.value[key];
}

function drop(ev: any) {
    isDragging.value = false;
    ev.preventDefault();
    const tgtId = ev.target.parentElement.id;
    const srcId = currentElement.sourceId
    var newData = null;
    var slot = null;
    var idx = null;
    console.log(currentElement, tgtId);

    idx = parseInt(srcId.slice(-1)) - 1;
    slot = getSlotById(srcId);

    if (tgtId.slice(0, 4) === "src_") {
        if ( ! isIndicesMatchingByColor(tgtId, currentElement.draggableId)) {
            console.log("Colors not matching!")
            return false;
        }
        newData = getData(slot, idx);
        idx = parseInt(tgtId.slice(-1)) - 1;
        slot = getSlotById(tgtId);
        console.log(newData, slot, idx)
        slot[idx] = newData;
        return
    }

    newData = getData(slot, idx);

    if (tgtId[0] === "p" && tgtId.length === 3) {
        slot = getSlotById(tgtId);
        idx = parseInt(tgtId.slice(-1)) - 1;
        setPlayerDice(slot, newData);
    }

    if (tgtId === "el_1") {
        slot = getSlotById(tgtId);
        slots.value.el = [newData];
    }
}

function randomInt(min: number, max: number): number {
    return Math.floor(Math.random() * (max - min + 1) + min);
}

function rollDice() {
    for (const key of ["bl", "gn", "or", "pr"]) {
        for (const el of slots.value[key]) {
            if (el) {
                el.value = key + randomInt(1, 6);
            }
        }
    }
}

function resetDice() {
    slots.value = deepCopy(initialSlots);
}

function playerNeedsDropSlot(p: any) {
    if (!p[0]) return true;
    if (!p[1] && playersCount == 2) return true;
    return false;
}

function stopDrag() {
    isDragging.value = false;
}

rollDice();
</script>

<template>
    <div class="main">
        <h1>&nbsp;</h1>

        <div class="row mb-3">
            <div class="col">
                <h2>Elephant</h2>
                <div class="player-body" id="el_1">
                    <div v-if="slots.el[0]"
                        :id="slots.el[0].id"
                        class="btn btn-default dice-cont"
                        :class="'dice-' + slots.el[0].value"
                        draggable="true"
                        :ondragstart="drag"
                    ></div>
                    <div v-if="!slots.el[0]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                </div>
            </div>
        </div>

        <div class="row">
            <div class="col">
                <div class="row">
                    <div class="col">
                        <div class="player-cont">
                            <label class="player-label player1">Player 1</label>
                            <div class="player-body" id="p1_">
                                <div id="p1_1">
                                    <div v-if="slots.p1[0]"
                                        :id="slots.p1[0].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p1[0].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div id="p1_2">
                                    <div v-if="slots.p1[1]"
                                        :id="slots.p1[1].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p1[1].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div v-if="playerNeedsDropSlot(slots.p1)" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="row" v-if="playersCount > 2">
                    <div class="col">
                        <div class="player-cont">
                            <label class="player-label player3">Player 3</label>
                            <div class="player-body" id="p3_">
                                <div id="p3_1">
                                    <div v-if="slots.p3[0]"
                                        :id="slots.p3[0].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p3[0].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div id="p3_2">
                                    <div v-if="slots.p3[1]"
                                        :id="slots.p3[1].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p3[1].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div v-if="playerNeedsDropSlot(slots.p3)" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="col-6 js-rollable-dice">
                <div class="row mb-2">
                    <div class="col">
                        <div id="src_bl1">
                            <div v-if="slots.bl[0]"
                                :id="slots.bl[0].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.bl[0].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.bl[0]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                    <div class="col">
                        <div id="src_bl2">
                            <div v-if="slots.bl[1]"
                                :id="slots.bl[1].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.bl[1].value"
                                draggable="true"
                                :ondragstart="drag"
                                :ondragend="stopDrag"
                            ></div>
                            <div v-if="!slots.bl[1]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                </div>

                <div class="row mb-2">
                    <div class="col">
                        <div id="src_gn1">
                            <div v-if="slots.gn[0]"
                                :id="slots.gn[0].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.gn[0].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.gn[0]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                    <div class="col">
                        <div id="src_gn2">
                            <div v-if="slots.gn[1]"
                                :id="slots.gn[1].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.gn[1].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.gn[1]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                </div>

                <div class="row mb-2">
                    <div class="col">
                        <div id="src_or1">
                            <div v-if="slots.or[0]"
                                :id="slots.or[0].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.or[0].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.or[0]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                    <div class="col">
                        <div  id="src_or2">
                            <div v-if="slots.or[1]"
                                :id="slots.or[1].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.or[1].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.or[1]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                </div>

                <div class="row mb-2">
                    <div class="col">
                        <div id="src_pr1">
                            <div v-if="slots.pr[0]"
                                :id="slots.pr[0].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.pr[0].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.pr[0]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                    <div class="col">
                        <div id="src_pr2">
                            <div v-if="slots.pr[1]"
                                :id="slots.pr[1].id"
                                class="btn btn-default dice-cont"
                                :class="'dice-' + slots.pr[1].value"
                                draggable="true"
                                :ondragstart="drag"
                            ></div>
                            <div v-if="!slots.pr[1]" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                        </div>
                    </div>
                </div>

                <div class="row">
                    <div class="col gy-3">
                        <button class="btn btn-lg btn-primary"
                            v-on:click="rollDice()" style="margin-right: 10px;">Roll</button>
                        <button class="btn btn-lg btn-danger"
                            v-on:click="resetDice()">Reset</button>
                    </div>
                </div>

                <div class="row">
                    <div class="col gy-3">
                        <a href="javascript:void(0)" v-on:click="resetDice(); $emit('newGame')">New game</a>
                    </div>
                </div>
            </div>

            <div class="col">
                <div class="row">
                    <div class="col">
                        <div class="player-cont">
                            <label class="player-label player1">Player 2</label>
                            <div class="player-body" id="p2_">
                                <div id="p2_1">
                                    <div v-if="slots.p2[0]"
                                        :id="slots.p2[0].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p2[0].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div id="p2_2">
                                    <div v-if="slots.p2[1]"
                                        :id="slots.p2[1].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p2[1].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div v-if="playerNeedsDropSlot(slots.p2)" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="row" v-if="playersCount > 3">
                    <div class="col">
                        <div class="player-cont">
                            <label class="player-label player3">Player 4</label>
                            <div class="player-body" id="p4_">
                                <div id="p4_1">
                                    <div v-if="slots.p4[0]"
                                        :id="slots.p4[0].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p4[0].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div id="p4_2">
                                    <div v-if="slots.p4[1]"
                                        :id="slots.p4[1].id"
                                        class="btn btn-default dice-cont"
                                        :class="'dice-' + slots.p4[1].value"
                                        draggable="true"
                                        :ondragstart="drag"
                                    ></div>
                                </div>
                                <div v-if="playerNeedsDropSlot(slots.p4)" class="player-dice-welcome" :ondrop="drop" :ondragover="allowDrop"></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
