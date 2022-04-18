<template>
    <svg 
        :x="pos.x" 
        :y="pos.y">
        <sub-node v-for="subnode in subnodes" :key="subnode.id" :ref="subnode.id" :x="subnode.x" :y="subnode.y" :width="subnode.width" :height="subnode.height" :text="subnode.text" @onMouseDown="onMouseDown" @onMouseUp="onMouseUp" @onStartLink="onStartLink" @onEndLink="onEndLink"/>
    </svg>
</template>

<script>
import SubNode from './SubNode.vue';
import { v4 as uuidv4 } from 'uuid';

export default {
    name: 'node',
    props: {
        x: {
            type: Number,
            default: 0
        },
        y: {
            type: Number,
            default: 0
        },
        subs: {
            type: Array,
            default: function () {
                return []
            }
        }
    },
    data() {
        return {
            pos: {
                x: 0,
                y: 0
            },
            subnodes: [],
            yspan: 20,
            selected: false
        };
    },
    updated() {
        this.subnodes.forEach((subnode) => {
            subnode.component = this.$refs[subnode.id][0];
        });
    },
    mounted() {
        console.log("Node mounted");
        this.pos.x = this.x;
        this.pos.y = this.y;

        this.clearSubNodes();
        for (let i = 0; i < this.subs.length; ++i) {
            this.addSubNode(this.subs[i], this.pos.x, this.pos.y);
        }
        console.log(this.pos);
    },
    methods: {
        onMouseDown(subnode, pos) {
            this.selected = true;
            this.$emit(
                "onStartDrag",
                this,
                subnode,
                pos.x - this.pos.x,
                pos.y - this.pos.y
            )
        },

        onMouseUp() {
            console.log("Node mouseDown");
            this.$emit("onEndDrag");
        },

        unselect() {
            this.selected = false;
            for (let i = 0; i < this.subnodes.length; ++i) {
                if (this.subnodes[i].component != undefined) {
                    this.subnodes[i].component.unselect();
                }
            }
        },

        onStartLink(subnode, port) {
            console.log("Node onStartLink");
            this.$emit("onStartLink", this, subnode, port);
        },

        onEndLink(subnode, port) {
            console.log("Node onEndLink");
            this.$emit("onEndLink", this, subnode, port);
        },

        move(x, y) {
            console.log("node move " + this.pos.x + ", " + this.pos.y);
            this.pos.x += x;
            this.pos.y += y;
        },

        addSubNode(text, parentX, parentY) {
            var subnode = {
                id: `${uuidv4()}`,
                x: parentX,
                y: parentY + (this.subnodes.length * this.yspan),
                width: 100,
                height: 20,
                text: text
            };

            console.log("addsubnode: " + subnode.text + "; y:" + subnode.y);
            this.subnodes.push(subnode);
        },

        clearSubNodes() {
            while (this.subnodes.length > 0) {
                this.subnodes.pop();
            }
        }
    },
    components: {
        'sub-node': SubNode
    }
}
</script>