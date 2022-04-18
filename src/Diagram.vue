<template>
    <div style="width: 500px; height: 500px; border:1px solid black;">
        <svg
            width="500"
            height="500"
            :style="{backgroundColor:bgColor, marginTop:y, marginLeft:x}"
            @mousedown="mouseDown"
            @mousemove="mouseMove"
            @mouseup="mouseUp">
            <node v-for="node in nodes" :key="node.id" :ref="node.id" :x="node.x" :y="node.y" :subs="node.subs" @onStartDrag="onStartDrag" @onEndDrag="onEndDrag" @onStartLink="onStartLink" @onEndLink="onEndLink"/>
            <node-link v-for="link in links" :key="link.id" :ref="link.id" :startX="link.startX" :startY="link.startY" :endX="link.endX" :endY="link.endY" />
        </svg>
    </div>
</template>

<script>
// import SvgPanZoom from 'vue-svg-pan-zoom';
import Node from './Node.vue';
import Link from './Link.vue';
import { v4 as uuidv4 } from 'uuid';

function removeItemFromArray(arr, item) {
    for (var i = 0; i < arr.length; ++i) {
        if (arr[i] === item) {
            arr.splice(i, 1);
            i--;
        }
    }
}

export default {
    name: 'diagram',
    props: {
        x: {
            type: Number,
            default: 0
        },
        y: {
            type: Number,
            default: 0
        }
    },
    data() {
        return {
            nodes: [
                {
                    id: '0',
                    x: 0,
                    y: 0,
                    subs: [
                        'test A',
                        'test B'
                    ],
                    component: undefined
                },
                {
                    id: '1',
                    x: 200,
                    y: 100,
                    subs: [
                        'test C',
                        'test D'
                    ],
                    component: undefined
                }
            ],
            links: [],
            bgColor: 'white',
            panEnabled: true,
            startDrag: false,
            startLink: false,
            selectedNode: undefined,
            selectedSubNode: undefined,
            draggedItem: undefined,
            selectedLink: undefined,
            lastMouseX: 0,
            lastMouseY: 0,
            zoom: 1,
        }
    },
    updated() {
        this.nodes.forEach((node) => {
            node.component = this.$refs[node.id][0];
        });

        this.links.forEach((link) => {
            link.component = this.$refs[link.id][0];
        });
    },
    components: {
        // SvgPanZoom,
        Node,
        'node-link': Link
    },
    methods: {
        onStartDrag(node, subnode, x, y) {
            console.log("onStartDrag");
            this.panEnabled = false;
            
            if (!this.startDrag && this.selectedNode) {
                this.selectedNode.unselect();
                this.selectedNode = undefined;
                this.selectedSubNode = undefined;
            }

            if (node) {
                this.selectedNode = this.draggedItem = node;
            }
            if (subnode) {
                this.selectedSubNode = subnode;
            }
            console.log(`${x}, ${y} ${this.panEnabled}`);
            this.startDrag = true;
        },
        onEndDrag() {
            console.log("onEndDrag");
            this.panEnabled = true;
            this.startDrag = false;
            this.draggedItem = undefined;
        },
        onStartLink(node, subnode, port) {
            console.log(port);
            this.panEnabled = false;
            this.startLink = true;
            console.log("subnode.pos: " + subnode.pos.x + ", " + subnode.pos.y);
            this.selectedLink = {
                id: `${uuidv4()}`,
                startX: node.pos.x + subnode.pos.x + port.pos.x,
                startY: node.pos.y + subnode.pos.y + port.pos.y,
                endX: node.pos.x + subnode.pos.x + port.pos.x,
                endY: node.pos.y + subnode.pos.y + port.pos.y,
                nodeOutput: node,
                subNodeOutput: subnode
            };
            this.links.push(this.selectedLink);
        },
        onEndLink(node, subnode, port) {
            if (this.startLink && this.selectedLink) {
                this.selectedLink.nodeInput = node;
                this.selectedLink.subNodeInput = subnode;
                console.log(`Before ${this.selectedLink.endX}`);
                if (this.selectedLink.component) {
                    this.selectedLink.component.setEnd(node.pos.x + subnode.pos.x + port.pos.x, node.pos.y + subnode.pos.y + port.pos.y);
                }
                this.startLink = false;
            }
        },
        beforePan() {
            if (this.panEnabled) {
                if (this.selectedNode) {
                    this.selectedNode.unselect();
                    this.selectedNode = undefined;
                }
                this.selectedSubNode = undefined;
            }
            return this.panEnabled;
        },
        mouseDown(pos) {
            console.log(`Mouse Down on ${pos.x}, ${pos.y}`);
            if (!this.startDrag && this.selectedNode) {
                this.selectedNode.unselect();
                this.selectedNode = undefined;
                this.selectedSubNode = undefined;
            }
        },
        mouseMove(pos) {
            if (this.startDrag && this.draggedItem) {

                let x = (pos.x - this.lastMouseX) / this.zoom;
                let y = (pos.y - this.lastMouseY) / this.zoom;
                this.draggedItem.move(x, y);

                for (let i = 0; i < this.links.length; ++i) {
                    let linkOutput;
                    let linkInput;
                    if (!linkOutput && this.links[i].nodeOutput === this.selectedNode) {
                        linkOutput = this.links[i];
                    }
                    if (!linkInput && this.links[i].nodeInput === this.selectedNode) {
                        linkInput = this.links[i];
                    }

                    if (linkOutput) {
                        linkOutput.component.moveStart(x, y);
                    } 
                    if (linkInput) {
                        linkInput.component.moveEnd(x, y);
                    }
                }

            } else if (this.startLink && this.selectedLink) {
                if (this.selectedLink.component) {
                    let x = (pos.x - this.lastMouseX) / this.zoom;
                    let y = (pos.y - this.lastMouseY) / this.zoom;
                    this.selectedLink.component.moveEnd(x, y);
                } else {
                    this.startLink = false;
                    this.selectedLink = undefined;
                }
            }

            this.lastMouseX = pos.x;
            this.lastMouseY = pos.y;
        },
        mouseUp() {
            if (this.startLink && this.selectedLink) {
                removeItemFromArray(this.links, this.selectedLink);
            }

            this.draggedItem = undefined;
            this.panEnabled = true;
            this.startDrag = false;
            this.startLink = false;
        },
        onZoom(zoom) {
            console.log(zoom);
            this.zoom = zoom;
        },
        addNode(x, y, subs) {
            var newNode = {
                id: `${uuidv4()}`,
                x: x,
                y: y,
                subs: subs,
                component: undefined
            };
            console.log('nodes: ' + this.nodes.length);
            this.nodes.push(newNode);
        },
        clearNodes() {
            while (this.nodes.length > 0) {
                this.nodes.pop();
            }
            while (this.links.length > 0) {
                this.links.pop();
            }
        }
    }
}
</script>