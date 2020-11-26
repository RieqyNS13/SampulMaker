<template>
<div style="width:100%">
    <slot name="loading" v-if="loading"></slot>
    <canvas id="myCanvas" style="width:100%"></canvas>
    <slot name="between"></slot>
    <div v-for="(item, n) in itemsData" :key="n">
        <div v-if="selectedContainer && selectedContainer.index==n">
            <div v-for="(config, i) in configs" :key="i">
                <div>
                    <slot :name="config.name" :item="item">
                    </slot>
                </div>

            </div>

        </div>
    </div>
</div>
</template>

<script>
import {
    Stage,
    Shape,
    Bitmap,
    Container,
    Text,
    Touch
} from "@createjs/easeljs";
var isLoaded = false;
export default {
    props: {
        items: Array,
        img: String,
        isMounted: {
            type: Boolean,
            default: function () {
                return false
            }
        },
        configs: {
            type: Array,
            default: function () {
                return [{
                        name: 'color',
                        label: 'Warna teks'
                    }, {
                        name: 'fontfamily',
                        label: 'Jenis teks'
                    }, {
                        name: 'size',
                        label: 'Ukuran teks'
                    },

                ]
            }
        }
    },
    created() {
        this.stage = this.bitmap = null;
        isLoaded = false;
        this.image = this.img;
        if (this.items) {
            this.itemsData = items
        }
    },
    watch: {
        itemsData: {
            handler: function (newVal, oldVal) {
                if (!this.isInitialized) return;
                if (!this.bitmap.getTransformedBounds()) return;
                console.log('new');
                console.log(newVal);
                console.log('old');
                console.log(oldVal);
                console.log(this.containers);
                console.log(this.bitmap)
                //if (!bitmap) return;

                let previousContainerData = [];
                let removedChildIndex = [];
                for (let i = 1; i < this.stage.numChildren; i++) {
                    previousContainerData.push({
                        x: this.stage.getChildAt(i).x,
                        y: this.stage.getChildAt(i).y
                    });
                    removedChildIndex.push(i)
                    //previousContainerData.y=stage.getChildAt(i).y;
                }
                //console.log(previousContainerData);
                this.stage.removeAllEventListeners();
                this.stage.removeChildAt(...removedChildIndex)
                //alert('hapus')
                this.containers = [];
                newVal.forEach((item, item_index) => {
                    // if (item.text) item.text = item.text.toString();
                    // else item.text = '';
                    this.createText(this.stage, this.bitmap, item, item_index, previousContainerData[item_index])
                    console.log('watch ' + item_index)
                    //alert('cok')
                });
                this.stage.update();
            },
            deep: true
        }
    },
    data() {
        return {
            isInitialized: false,
            stage: null,
            bitmap: null,
            image: null,
            containers: [],
            selectedContainer: null,
            loading: false,
            itemsData: [],

        }
    },
    mounted() {
        //console.log(this.configs)
        console.log('mounted sampul maker BEGIN')
        console.log(this.itemsData)
        if (!this.isMounted) return;
        this.loading = true;

        this.stage = new Stage("myCanvas");
        Touch.enable(this.stage);
        //        createjs.Touch.enable(stage);
        //stage.mouseMoveOutside = true;
        let img = new Image;
        this.bitmap = new Bitmap(img);
        img.onload = () => {
            this.bitmap.setTransform(0, 0, 1, 1);
            //stage section
            this.stage.addChild(this.bitmap);
            this.stage.canvas.width = this.bitmap.getTransformedBounds().width;
            this.stage.canvas.height = this.bitmap.getTransformedBounds().height;

            //section buat text & rect
            this.itemsData.forEach((item, item_index) => {
                // if (item.text) item.text = item.text.toString();
                // else item.text = '';
                this.createText(this.stage, this.bitmap, item, item_index)
            });
            isLoaded = img.complete && img.naturalHeight !== 0;
            this.stage.update();
            console.log('mounted sampul maker END')

        }
        img.crossOrigin = "Anonymous"
        img.src = this.image;
    },
    methods: {
        initialize() {
            console.log('BEGIN initialize sampul maker')
            return new Promise((resolve, reject) => {
                this.loading = true;
                this.stage = new Stage("myCanvas");
                Touch.enable(this.stage);
                let img = new Image;
                this.bitmap = new Bitmap(img);

                img.onload = () => {
                    this.bitmap.setTransform(0, 0, 1, 1);
                    //stage section
                    this.stage.addChild(this.bitmap);

                    this.stage.canvas.width = this.bitmap.getTransformedBounds().width;
                    this.stage.canvas.height = this.bitmap.getTransformedBounds().height;

                    //section buat text & rect
                    this.containers = [];
                    this.itemsData.forEach((item, item_index) => {

                        this.createText(this.stage, this.bitmap, item, item_index)
                        console.log('initialize ' + item_index)
                    });
                    isLoaded = img.complete && img.naturalHeight !== 0;

                    this.loading = false;
                    this.stage.update();
                    console.log('END initialize sampul maker')
                    this.isInitialized = true;
                    resolve(this.stage)
                }
                img.crossOrigin = "Anonymous"
                img.src = this.image;
                //console.log('img src ' + this.image)
            })
        },
        resetContainers() {
            let removedChildIndex = [];
            for (let i = 1; i < this.stage.numChildren; i++) {
                removedChildIndex.push(i)
            }
            this.stage.removeAllEventListeners();
            this.stage.removeChildAt(...removedChildIndex)
            this.containers = [];
        },
        //merefresh canvas jika data items berubah
        refreshCanvas() {
            this.resetContainers();
            this.itemsData.forEach((item, item_index) => {
                this.createText(this.stage, this.bitmap, item, item_index)
            });
        },
        createText: function (stage, bitmap, objText, objTextIndex, containerData = {}) {
            let fontsize1 = bitmap.getTransformedBounds().width / (10 * (10 / objText.size));
            //console.log('fontsize: ' + fontsize1)
            let x;
            if (!objText.x) {
                let x_center = bitmap.getTransformedBounds().width / 2;
                //jika ada atribut x_append, maka nilai x_center akan ditambah dengan x_append
                if (objText.x_append) x_center += objText.x_append;
                x = x_center;
            } else {
                x = objText.x
            }
            var container = new Container();
            container.x = containerData.x ? containerData.x : x;
            container.y = containerData.y ? containerData.y : objText.y;

            const maxTextWidth = bitmap.getTransformedBounds().width - 10;
            let objLineArray = this.wrapText(container, objText, 0, 0, maxTextWidth, fontsize1, fontsize1); //

            let alpha = 0.01;
            if (this.selectedContainer && this.selectedContainer.index == objTextIndex) alpha = 0.3
            container.addChild(this.rectText(objLineArray, fontsize1, alpha))

            for (let i = 0; i < objLineArray.length; i++) {
                container.addChild(objLineArray[i])
            }
            container.index = objTextIndex;
            this.containers.push(container)

            stage.addChild(container);
            container.on("pressmove", (evt) => {

                // currentTarget will be the container that the event listener was added to:
                evt.currentTarget.x = evt.stageX;
                evt.currentTarget.y = evt.stageY;
                // make sure to redraw the stage to show the change:
                stage.update();
            });
            container.on("mousedown", (evt) => {
                //console.log(container.index)
                this.containers.forEach(item => {
                    item.getChildAt(0).alpha = 0.01;
                });
                container.getChildAt(0).alpha = 0.3;
                this.selectedContainer = container;
                stage.update();
                // this.selectedContainer = evt;
            });
            stage.update();
        },
        rectText: function (objLineArray, fontsize, alpha = 0.01) {
            let maxTextWidth = 0;
            let sumTextHeight = 0;
            for (let i = 0; i < objLineArray.length; i++) {
                const objLineWidth = objLineArray[i].getMeasuredWidth();
                if (objLineWidth > maxTextWidth) maxTextWidth = objLineWidth;
            }
            var rect = new Shape();
            rect.alpha = alpha;
            const x = -1 * (maxTextWidth / 2)
            //alert(maxTextWidth)
            //console.log('maxTextWidth: ' + maxTextWidth)
            rect.graphics.beginFill("rgb(255,255,255)").drawRect(x, 0, maxTextWidth, objLineArray.length * fontsize);
            this.rect = {
                x: x
            };
            return rect;
        },
        wrapText: function (stage, objText, x, y, maxWidth, lineHeight, fontsize = 100) {
            let objLineArray = [];
            let text = objText.text ? objText.text.toString().replace(/\s\s+/g, ' ') : ''; //hapus semua double spasi/tab menjadi 1 spasi
            var words = text.split(' ');
            var line = '';
            var objLine;
            var objTestLine;
            const fontStyle = `${fontsize}px '${objText.fontfamily?objText.fontfamily:'Arial'}'`;

            for (var n = 0; n < words.length; n++) {
                var testLine = line + words[n] + ' ';
                objTestLine = new Text(testLine, fontStyle, objText.color);
                var testWidth = objTestLine.getMeasuredWidth();
                // objTestLine.x=x;
                // objTestLine.y=y;
                if (testWidth > maxWidth && n > 0) {

                    objLine = new Text(line, fontStyle, objText.color);
                    objLine.x = x;
                    objLine.y = y;
                    objLine.textBaseline = "top";
                    objLine.textAlign = "center";
                    objLineArray.push(objLine)
                    //stage.addChild(objLine);
                    //context.fillText(line, x, y);
                    line = words[n] + ' ';
                    y += lineHeight;
                } else {
                    line = testLine;
                }
            }
            objLine = new Text(line, fontStyle, objText.color);
            objLine.x = x;
            objLine.y = y;
            objLine.textBaseline = "top";
            objLine.textAlign = "center";
            objLineArray.push(objLine)
            //stage.addChild(objLine);
            //context.fillText(line, x, y);
            return objLineArray;
        },
        getContainers() {
            return this.containers;
        },
        getItems() {
            return this.itemsData;
        },
        toDataURL() {
            //menjadikan alpha=0.3 menjadi alpha=0 pada semua kontainer
            this.containers.forEach(container=>{
                container.getChildAt(0).alpha = 0;
            });
            this.stage.update();
            return this.stage.toDataURL();
        },
        isLoading() {
            return this.loading;
        },
        setImage(img) {
            this.isInitialized = false;
            this.image = img;
        },
        setItems(items) {
            this.isInitialized = false;
            this.itemsData = items
        }
    },
}
</script>
