<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
    data() {
        return {
            isTouch: false,
            hovering: false,
            active: false
        };
    },
    methods: {
        hover() {
            if (this.isTouch) return;
            this.hovering = true;
        },
        unhover() {
            this.hovering = false;
        },
        activate() {
            this.active = true;
        },
        deactivate() {
            this.active = false;
        },
        exit() {
            this.unhover();
            this.deactivate();
        }
    }
});
</script>

<template>
    <button @touchstart="isTouch = true; activate();" @mousedown="activate" @touchend="deactivate" @mouseup="deactivate" @mouseenter="hover" @mouseleave="exit" :class="{ hover: hovering, active: active }">
        <slot></slot>
    </button>
</template>

<style scoped lang="scss"></style>