<template>
    <my-page title="字幕拼接" :page="page">
        <div class="empty-box" v-if="!images.length">
            <div class="text">请点击 “+” 添加图片</div>
        </div>
        <ul class="img-list">
            <li class="item" v-for="item, index in images">
                <img class="img" :src="item.src" ondragstart='return false;'>
                <div v-if="index !== 0">
                </div>
                <div class="mask-top" :style="{bottom: (item.bottom + item.height) + 'px'}"></div>
                <div class="mask-bottom" :style="{height: item.bottom + 'px'}"></div>
                <div class="line" 
                    @mousedown="handlerMouseDown(item, $event)"
                    :style="{bottom: (item.bottom + item.height) + 'px'}"></div>
                <div
                    @mousedown="handlerMouseDown2(item, $event)"
                    class="line" :style="{bottom: item.bottom + 'px'}"></div>
                <div class="btns">
                    <ui-icon-button class="btn up" icon="arrow_upward" @click="upward(item, index)" v-if="index !== 0" />
                    <ui-icon-button class="btn down" icon="arrow_downward" @click="downward(item, index)" v-if="index !== images.length - 1" />
                    <ui-icon-button class="btn remove" icon="close" @click="remove(item, index)" />
                </div>
            </li>
        </ul>
        <!-- <ui-raised-button class="file-select-btn" label="选择图片" primary>
        </ui-raised-button> -->
        <!-- <ui-raised-button primary label="添加" /> -->
        <div class="float-btn-box">
            <ui-float-button class="btn btn-make" secondary icon="send" 
                title="拼接截图" @click="preview" v-if="images.length > 1"></ui-float-button>
            <ui-float-button class="btn btn-file" title="添加图片">
                <ui-icon value="image" />
                <input type="file" class="ui-file-button" multiple="multiple" accept="image/*" @change="fileChange($event)">
            </ui-float-button>
            <ui-float-button class="btn btn-movie" title="上传视频获取截图">
                <ui-icon value="movie" />
                <input class="ui-file-button" id="file" type="file" @change="fileChange2($event)" />
            </ui-float-button>
            <ui-float-button class="btn btn-make" secondary icon="settings" title="设置" @click="toggleSetting"></ui-float-button>
        </div>
        <!-- <ui-raised-button class="" primary label="上传视频获取截图">
            <input class="ui-file-button" id="file" type="file" @change="fileChange2($event)" />
        </ui-raised-button> -->

        <!-- <button @click="preview">上传视频获取截图</button> -->
        <ui-drawer class="preview-box" :docked="false" right :open="previewVisible" @close="togglePreview()">
            <ui-appbar title="预览">
                <ui-icon-button icon="close" slot="left" @click="togglePreview" title="关闭" />
                <ui-icon-button icon="file_download" slot="right" @click="download" title="下载" />
            </ui-appbar>
            <div class="body">
                <canvas id="canvas"></canvas>
            </div>
        </ui-drawer>
        <ui-drawer class="setting-box" :docked="false" right :open="settingVisible" @close="toggleSetting()">
            <ui-appbar title="设置">
                <ui-icon-button icon="close" slot="left" @click="toggleSetting" title="关闭" />
            </ui-appbar>
            <div class="body">
                <ui-text-field v-model.number="height" label="默认字幕高度（px）" />
                <br>
                <ui-text-field v-model.number="bottom" label="默认底部距离（px）" />
                <br>
                <ui-text-field v-model.number="offset" label="截图间距（px）" />
            </div>
        </ui-drawer>
        <!-- 视频 -->
        <!-- <input class="file" id="file" type="file" @change="fileChange2" /> -->
        <div v-if="loaded">
            <video id="player" controls></video>
            <br>
            <ui-raised-button class="capture" label="截图" primary @click="capture" />
        </div>
        <canvas id="canvas" class="canvas" style="display: none"></canvas>
    </my-page>
</template>

<script>
    const saveAs = window.saveAs
    const maxWidth = 800

    export default {
        data () {
            return {
                bottom: 0,
                height: 60,
                offset: 0,
                previewVisible: false,
                images: [
                    // {
                    //     src: 'https://cdn.v2ex.com/avatar/9e0f/9113/14640_normal.png?m=1443150294',
                    //     bottom: 0,
                    //     height: 100
                    // },
                    // {
                    //     src: 'https://www.baidu.com/img/bd_logo1.png?where=super',
                    //     bottom: 0,
                    //     height: 100
                    // },
                    // {
                    //     src: 'https://www.baidu.com/img/bd_logo1.png?where=super',
                    //     bottom: 0,
                    //     height: 100
                    // }
                ],
                // 视频
                loaded: false,
                // 设置
                settingVisible: false,
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'settings',
                            click: this.toggleSetting,
                            title: '设置'
                        },
                        {
                            type: 'icon',
                            icon: 'help',
                            to: '/join/help',
                            title: '帮助'
                        }
                    ]
                }
            }
        },
        mounted() {
            this.addImage('https://img1.yunser.com/avatar/avatar_1.jpg')
            // this.addImage('https://img1.yunser.com/avatar/avatar_1.jpg')
            // this.addImage('https://www.baidu.com/img/bd_logo1.png?where=super')
            // this.addImage('https://cdn.v2ex.com/avatar/9e0f/9113/14640_normal.png?m=1443150294')
            // this.addImage('https://cdn.v2ex.com/avatar/9e0f/9113/14640_normal.png?m=1443150294')
            // setTimeout(() => {
            //     this.preview()
            // }, 2000)
        },
        methods: {
            toggleSetting() {
                this.settingVisible = !this.settingVisible
            },
            download() {
                let canvas = document.getElementById('canvas')
                canvas.toBlob(function(blob) {
                    saveAs(blob, 'pretty image.png')
                })
            },
            togglePreview() {
                this.previewVisible = !this.previewVisible
            },
            handlerMouseDown(item, e) {
                // let isDown = true
                let mouseMove
                let mouseUp
                let downY = e.pageY
                let originHeight = item.height
                document.addEventListener('mousemove', mouseMove = e => {
                    console.log(e.pageY - downY)
                    let height = originHeight + (downY - e.pageY)
                    if (item.bottom + height <= item.displayHeight) {
                        item.height = height
                    }
                })
                document.addEventListener('mouseup', mouseUp = e => {
                    document.removeEventListener('mousemove', mouseMove)
                    document.removeEventListener('mouseup', mouseUp)
                })
            },
            handlerMouseDown2(item, e) {
                // let isDown = true
                let mouseMove
                let mouseUp
                let downY = e.pageY
                let originBottom = item.bottom
                let originHeight = item.height
                document.addEventListener('mousemove', mouseMove = e => {
                    console.log(e.pageY - downY)
                    let bottom = originBottom + (downY - e.pageY)
                    let height = originHeight - (item.bottom - originBottom)
                    console.log(bottom + height, item.displayHeight)
                    item.bottom = bottom
                    item.height = height
                    if (bottom + height <= item.displayHeight) {
                    }
                })
                document.addEventListener('mouseup', mouseUp = e => {
                    document.removeEventListener('mousemove', mouseMove)
                    document.removeEventListener('mouseup', mouseUp)
                })
            },
            handlerMouseMove(e) {
            },
            handlerMouseUp(e) {
                this.isDown = false
            },
            addImage(dataUrl) {
                let img = new Image()
                img.onload = () => {
                    let displayWidth = 400
                    let displayHeight = displayWidth * img.height / img.width
                    this.images.push({
                        id: '' + new Date().getTime(),
                        src: dataUrl,
                        image: img,
                        // bottom: 146,
                        // height: 120
                        bottom: this.bottom,
                        height: this.images.length === 0 ? displayHeight : this.height,
                        displayHeight: displayHeight
                    })
                    console.log('this.images', this.images)
                }
                img.src = dataUrl
            },
            drawImage(cxt, img, top) {
                return new Promise((resolve, reject) => {

                })
            },
            preview() {
                this.previewVisible = true
                this.$nextTick(() => {
                    this.preview2()
                })
            },
            preview2() {
                let previewWidth = maxWidth
                let totalHeight = 0
                let displayWidth = 400
                for (let i = 0; i < this.images.length; i++) {
                    let item = this.images[i]
                    let previewHeight = item.height * previewWidth / displayWidth
                    totalHeight += previewHeight + (i === 0 ? 0 : this.offset)
                }

                let canvas = document.getElementById('canvas')
                let ctx = canvas.getContext('2d')
                // canvas.style.width = displayWidth + 'px'
                // canvas.style.height = totalHeight + 'px'
                canvas.width = previewWidth
                canvas.height = totalHeight
                ctx.width = previewWidth
                ctx.height = totalHeight
                // console.log()
                let top = 0
                ctx.fillStyle = '#fff'
                ctx.fillRect(0, 0, previewWidth, totalHeight)
                // await this.drawImage(cxt, img, 0)
                for (let i = 0; i < this.images.length; i++) {
                    let item = this.images[i]
                    let img = this.images[i].image
                    // let height = previewWidth * img.height / img.width
                    let displayHeight = displayWidth * img.height / img.width
                    let previewHeight = item.height * previewWidth / displayWidth

                    let y = displayHeight - item.bottom - item.height
                    let yRate = y / displayHeight
                    let noMaskRate = item.height / displayHeight
                    console.log('item.height', item.height)
                    ctx.drawImage(img, 0, img.height * yRate, img.width, img.height * noMaskRate,
                        0, top, previewWidth, previewHeight)
                    top += previewHeight + (i === this.images.length - 1 ? 0 : this.offset)
                }
                // let img = new Image()
                // img.onload = () => {
                // }
                // img.src = this.images[0].src
                // ctx.fillRect(0, 0, 100, 100)
            },
            fileChange(e) {
                let files = e.target.files
                for (let file of files) {
                    // var file = e.target.files[0]
                    console.log(e.target.files)
                    var reader = new FileReader()
                    reader.onloadend = e => {
                        this.addImage(e.target.result)
                    }
                    reader.readAsDataURL(file)
                }
            },
            remove(item, index) {
                this.images.splice(index, 1)
            },
            upward(item, index) {
                this.images.splice(index, 1)
                this.images.splice(index - 1, 0, item)
            },
            downward(item, index) {
                this.images.splice(index, 1)
                this.images.splice(index + 1, 0, item)
            },
            fileChange2(e) {
                if (!e.target.files.length) {
                    return
                }
                let url = window.URL.createObjectURL(e.target.files[0])
                this.loaded = true
                this.$nextTick(() => {
                    let video = document.getElementById('player')
                    video.onload = () => {
                        let rect = video.getBoundingClientRect()
                        console.log(rect)
                        this.videoWidth = rect.width
                        this.videoHeight = rect.height
                    }
                    video.src = url
                })
            },
            capture() {
                let video = document.getElementById('player')
                let canvas = document.getElementById('canvas')
                let context = canvas.getContext('2d')

                let rect = video.getBoundingClientRect()
                console.log(rect)
                this.videoWidth = rect.width
                this.videoHeight = rect.height
                context.drawImage(video, 0, 0, this.videoWidth, this.videoHeight)
                this.addImage(canvas.toDataURL())
            }
        }
    }
</script>

<style lang="scss" scoped>
.editor-box {
    position: absolute;
    top: 0;
    left: 0;
    width: 50%;
    bottom: 0;
    padding: 16px;
    overflow: auto;
}
.preview-box {
    z-index: 100000;
    width: 100%;
    max-width: 450px;
    .body {
        position: absolute;
        top: 64px;
        left: 0;
        right: 0;
        bottom: 0;
        padding: 16px;
        overflow: auto;
    }
    canvas {
        width: 400px;
        border: 1px solid #ccc;
    }
}
.setting-box {
    z-index: 100000;
    width: 100%;
    max-width: 320px;
    .body {
        position: absolute;
        top: 64px;
        left: 0;
        right: 0;
        bottom: 0;
        padding: 16px;
        overflow: auto;
    }
}
.img-list {
    width: 400px;
    // margin: 0 auto;
    .item {
        position: relative;
        margin-bottom: 16px;
        border: 1px solid #999;
        user-select: none;
    }
    .img {
        display: block;
        width: 100%;
        user-select: none;
    }
    .mask-top {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 100px;
        background-color: rgba(0, 0, 0, .6);
    }
    .mask-bottom {
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        height: 30px;
        background-color: rgba(0, 0, 0, .6);
    }
    .line {
        position: absolute;
        left: -16px;
        bottom: 4px;
        right: -16px;
        height: 4px;
        background-color: #09c;
        cursor: row-resize;
    }
    .btns {
        position: absolute;
        right: 0;
        top: 0;
        display: flex;
        height: 48px;
        .btn {
            color: #fff;
            // background-color: rgba(0, 0, 0, .1)
            text-shadow: 0 0 2px #333;
        }
    }
    // .remove {
    //     position: absolute;
    //     top: 0;
    //     right: 0;
    //     z-index: 10000000;
    //     color: #fff;
    // }
    // .up {
    //     position: absolute;
    //     top: 0;
    //     right: 48px;
    //     z-index: 10000000;
    //     color: #fff;
    // }
    // .down {
    //     position: absolute;
    //     top: 0;
    //     right: 96px;
    //     z-index: 10000000;
    //     color: #fff;
    // }
}
.float-btn-box {
    position: fixed;
    right: 24px;
    bottom: 24px;
    width: 48px;
    z-index: 1000;
    .btn {
        margin-top: 8px;
    }
}
// .btn-file {
//     position: fixed;
//     right: 24px;
//     bottom: 24px;
//     z-index: 1000;
// }
// .btn-make {
//     position: fixed;
//     right: 24px;
//     bottom: 24px + 72px;
//     z-index: 1000;
// }
// .btn-movie {
//     position: fixed;
//     right: 24px;
//     bottom: 24px + 72px;
//     z-index: 1000;
// }
.empty-box {
    padding: 80px 0;
    .text {
        color: #999;
        font-size: 16px;
        text-align: center;
    }
}
</style>
