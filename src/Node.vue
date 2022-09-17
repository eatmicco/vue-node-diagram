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
            selected: false,
            selectedSubNode: undefined,
        };
    },
    updated() {
        this.subnodes.forEach((subnode) => {
            subnode.component = this.$refs[subnode.id][0];
        });
    },
    mounted() {
        this.pos.x = this.x;
        this.pos.y = this.y;

        this.clearSubNodes();
        for (let i = 0; i < this.subs.length; ++i) {
            this.addSubNode(this.subs[i]);
        }
    },
    methods: {
        onMouseDown(subnode) {
            if (this.selectedSubNode != undefined && 
                this.selectedSubNode.text.localeCompare(subnode.text) != 0 &&
                this.selectedSubNode.component != undefined) {
                this.selectedSubNode.component.unselect();
            }

            var [s, i] = this.getSubNode(subnode.text);
            this.selectedSubNode = s;
            
            this.selected = true;
            this.$emit(
                "onStartDrag",
                this,
                subnode,
                i
            )
        },

        onMouseUp() {
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
            this.$emit("onStartLink", this, subnode, port);
        },

        onEndLink(subnode, port) {
            this.$emit("onEndLink", this, subnode, port);
        },

        move(x, y) {
            this.pos.x += x;
            this.pos.y += y;
        },

        addSubNode(text) {
            var subnode = {
                id: `${uuidv4()}`,
                x: this.pos.x,
                y: this.pos.y + (this.subnodes.length * this.yspan),
                width: 100,
                height: 20,
                text: text
            };

            this.subnodes.push(subnode);
        },

        clearSubNodes() {
            while (this.subnodes.length > 0) {
                this.subnodes.pop();
            }
        },

        getSubNode(text) {
            for (var i = 0; i < this.subnodes.length; i++) {
                var s = this.subnodes[i];
                if (s.text.localeCompare(text) == 0) {
                    return [s, i];
                }
            }
        }
    },
    components: {
        'sub-node': SubNode
    }
}
</script>