<template>
    <svg
        :x="pos.x" :y="pos.y">
        <port 
            :x="0" :y="portY"
            :size="portSize" @onMouseUp="onEndLink" />
        <rect 
            stroke="#555555"
            :fill="!selected ? '#FFFFFF' : '#AAAAAA'"
            stroke-width="1"
            :x="portSize" y="0"
            rx="3" ry="3"
            :width="width" :height="height"
            @mousedown="mouseDown"
            @mouseup="mouseUp" />
        <port 
            :x="outputX" :y="portY"
            :size="portSize" @onMouseDown="onStartLink"/>
        <text
            xml:space="preserve"
            style="font-style:normal;font-weight:normal;line-height:1.25;font-family:sans-serif;fill:#555555;fill-opacity:1;stroke:none;stroke-width:0.264583"
            :style="{fontSize:fontSize}"
            :x="textPosX"
            :y="textPosY"
            id="text30">{{ text }}</text>

    </svg>
</template>

<script>
import Port from './Port.vue';

export default {
    name: 'sub-node',
    props: {
        width: {
            type: Number,
            default: 72
        },
        height: {
            type: Number,
            default: 108
        },
        x: {
            type: Number,
            default: 0
        },
        y: {
            type: Number,
            default: 0
        },
        text: {
            type: String,
            default: "Name"
        }
    },
    data() {
        return {
            pos: {
                x: 0,
                y: 0
            },
            portSize: 4,
            fontSize: 10,
            selected: false
        };
    },
    mounted() {
        console.log("SubNode mounted");
        this.pos.x = this.x;
        this.pos.y = this.y;
        console.log(this.pos);
    },
    computed: {
        portY() {
            return (this.height/2) - (this.portSize/2);
        },
        outputX() {
            return this.width + this.portSize;
        },
        textPosX() {
            return this.portSize * 2;
        },
        textPosY() {
            return this.height - (this.fontSize / 2);
        }
    },
    methods: {
        mouseDown(_pos) {
            this.selected = true;
            this.$emit("onMouseDown", this, _pos);
        },

        mouseUp() {
            this.$emit("onMouseUp");
        },

        unselect() {
            this.selected = false;
        },

        onStartLink(port) {
            console.log("SubNode onStartLink " + this.pos.x + ", " + this.pos.y);
            this.$emit("onStartLink", this, port);
        },

        onEndLink(port) {
            console.log("SubNode onEndLink");
            this.$emit("onEndLink", this, port);
        },

        move(x, y) {
            console.log("subnode move: " + this.pos.x + ", " + this.pos.y);
            this.pos.x += x;
            this.pos.y += y;
        }
    },
    components: {
        Port
    }
}
</script>