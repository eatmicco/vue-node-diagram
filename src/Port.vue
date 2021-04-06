<template>
    <svg
        :x="x" :y="y"
        @mousedown="mouseDown"
        @mouseup="mouseUp"
        @mouseenter="mouseEnter"
        ref="svgPort">
        <rect 
            stroke="#555555"
            fill="#555555"
            stroke-width="1"
            x="0" y="0"
            :width="size" :height="size" />
    </svg>
</template>

<script>
export default {
    name: 'port',
    props: {
        x: {
            type: Number,
            default: 0
        },
        y: {
            type: Number,
            default: 0
        },
        size: {
            type: Number,
            default: 4
        }
    },
    methods: {
        mouseDown() {
            console.log('Port mouseDown');
            this.$emit("onMouseDown", 
                    {pos: {
                        x: this.x + (this.size/2),
                        y: this.y + (this.size/2)
                    }});
        },
        mouseUp() {
            console.log('Port mouseUp');
            this.$emit("onMouseUp",
                     {pos: {
                        x: this.x + (this.size/2),
                        y: this.y + (this.size/2)
                    }});
        },
        mouseEnter() {
            this.$refs.svgPort.style.cursor = "pointer";
        }
    }
}
</script>