<template>
    <svg>
        <path
            style="fill:none;stroke:#000000;stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;stroke-opacity:1"
            :d="d" />
    </svg>
</template>
<script>
export default {
    data() {
        return {
            start: {
                x: 0,
                y: 0
            },
            end: {
                x: 0,
                y: 0
            }
        };
    },
    props: {
        startX: {
            type: Number,
            default: 0
        },
        startY: {
            type: Number,
            default: 0
        },
        endX: {
            type: Number,
            default: 0
        },
        endY: {
            type: Number,
            default: 0
        }
    },
    mounted() {
        this.start.x = this.startX;
        this.start.y = this.startY;
        this.end.x = this.endX;
        this.end.y = this.endY;
    },
    computed: {
        d() {
            let lengthX = Math.abs(this.end.x - this.start.x);
            if (this.end.x < this.start.x) {
                let intersection1X = this.start.x + 10;
                let intersection1Y = this.start.y;
                let intersection2X = intersection1X;
                let lenghY = this.end.y - this.start.y;
                let intersection2Y = this.start.y + (lenghY * 0.5);
                let intersection3X = this.end.x - 10;
                let intersection3Y = intersection2Y;
                let intersection4X = intersection3X;
                let intersection4Y = this.end.y;
                return `M ${this.start.x},${this.start.y} L ${intersection1X},${intersection1Y} ${intersection2X},${intersection2Y} ${intersection3X},${intersection3Y} ${intersection4X},${intersection4Y} ${this.end.x},${this.end.y}`;                
            }
            let intersection1X = this.start.x + (lengthX * 0.5);
            let intersection1Y = this.start.y;
            let intersection2X = intersection1X;
            let intersection2Y = this.end.y;
            return `M ${this.start.x},${this.start.y} L ${intersection1X},${intersection1Y} ${intersection2X},${intersection2Y} ${this.end.x},${this.end.y}`;
        }
    },
    methods: {
        setStart(x, y) {
            this.start.x = x;
            this.start.y = y;
        },

        setEnd(x, y) {
            this.end.x = x;
            this.end.y = y;
        },

        moveStart(x, y) {
            this.start.x += x;
            this.start.y += y;
        },

        moveEnd(x, y) {
            this.end.x += x;
            this.end.y += y;
        }
    }
}
</script>