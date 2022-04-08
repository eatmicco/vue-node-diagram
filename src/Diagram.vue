<template>
    <div style="width: 500px; height: 500px; border:1px solid black;">
        <svg
            width="500"
            height="500"
            :style="{backgroundColor:bgColor, marginTop:y, marginLeft:x}"
            @mousedown="mouseDown"
            @mousemove="mouseMove"
            @mouseup="mouseUp">
            <node v-for="node in nodes" :key="'node_' + node.id" :ref="'node_' + node.id" :x="node.x" :y="node.y" :width="node.width" :height="node.height" :text="node.text" @onStartDrag="onStartDrag" @onEndDrag="onEndDrag" @onStartLink="onStartLink" @onEndLink="onEndLink"/>
            <node-link v-for="link in links" :key="'link_' + link.id" :ref="'link_' + link.id" :startX="link.startX" :startY="link.startY" :endX="link.endX" :endY="link.endY" />
        </svg>
    </div>
</template>

<script>
// import SvgPanZoom from 'vue-svg-pan-zoom';
import Node from './Node.vue';
import Link from './Link.vue';

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
                    text: 'node 1',
                    x: 0,
                    y: 0,
                    width: 100,
                    height: 20,
                    input: '',
                    output: '',
                    component: undefined
                },
                {
                    id: '1',
                    text: 'node 2',
                    x: 200,
                    y: 100,
                    width: 100,
                    height: 20,
                    input: '',
                    output: '',
                    component: undefined
                }
            ],
            links: [],
            bgColor: 'white',
            panEnabled: true,
            startDrag: false,
            startLink: false,
            selectedItem: undefined,
            draggedItem: undefined,
            selectedLink: undefined,
            lastMouseX: 0,
            lastMouseY: 0,
            zoom: 1,
        }
    },
    updated() {
        this.nodes.forEach((node) => {
            node.component = this.$refs['node_' + node.id][0];
        });

        this.links.forEach((link) => {
            link.component = this.$refs['link_' + link.id][0];
        });
    },
    components: {
        // SvgPanZoom,
        Node,
        'node-link': Link
    },
    methods: {
        onStartDrag(item, x, y) {
            console.log("onStartDrag");
            this.panEnabled = false;
            
            if (!this.startDrag && this.selectedItem) {
                this.selectedItem.unselect();
                this.selectedItem = undefined;
            }

            if (item) {
                this.selectedItem = this.draggedItem = item;
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
        onStartLink(node, port) {
            console.log(port);
            this.panEnabled = false;
            this.startLink = true;
            this.selectedLink = {
                id: `${this.links.length}`,
                startX: node.pos.x + port.pos.x,
                startY: node.pos.y + port.pos.y,
                endX: node.pos.x + port.pos.x,
                endY: node.pos.y + port.pos.y,
                nodeOutput: node
            };
            this.links.push(this.selectedLink);
        },
        onEndLink(node, port) {
            if (this.startLink && this.selectedLink) {
                this.selectedLink.nodeInput = node;
                console.log(`Before ${this.selectedLink.endX}`);
                if (this.selectedLink.component) {
                    this.selectedLink.component.setEnd(node.pos.x + port.pos.x, node.pos.y + port.pos.y);
                }
                this.startLink = false;
            }
        },
        beforePan() {
            if (this.panEnabled) {
                if (this.selectedItem) {
                    this.selectedItem.unselect();
                    this.selectedItem = undefined;
                }
            }
            return this.panEnabled;
        },
        mouseDown(pos) {
            console.log(`Mouse Down on ${pos.x}, ${pos.y}`);
            if (!this.startDrag && this.selectedItem) {
                this.selectedItem.unselect();
                this.selectedItem = undefined;
            }
        },
        mouseMove(pos) {
            if (this.startDrag && this.draggedItem) {

                let linkOutput;
                let linkInput;
                for (let i = 0; i < this.links.length; ++i) {
                    if (!linkOutput && this.links[i].nodeOutput === this.draggedItem) {
                        linkOutput = this.links[i];
                    }
                    if (!linkInput && this.links[i].nodeInput === this.draggedItem) {
                        linkInput = this.links[i];
                    }
                    if (linkOutput && linkInput) {
                        break;
                    }
                }

                let x = (pos.x - this.lastMouseX) / this.zoom;
                let y = (pos.y - this.lastMouseY) / this.zoom;
                console.log(`mouseMove ${x}, ${y}`);
                this.draggedItem.move(x, y);

                if (linkOutput) {
                    linkOutput.component.moveStart(x, y);
                } 
                if (linkInput) {
                    linkInput.component.moveEnd(x, y);
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
        addNode(text, x, y) {
            var newNode = {
                id: `${this.links.length}`,
                text: text,
                x: x,
                y: y,
                width: 100,
                height: 20,
                input: '',
                output: '',
                component: undefined
            };
            this.nodes.push(newNode);
        },
        clearNodes() {
            this.nodes.splice(0, this.nodes.length);
        }
    }
}
</script>