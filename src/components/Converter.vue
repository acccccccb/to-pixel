<script setup>
    import { Archive } from '@vicons/ionicons5';
    import { useMessage } from 'naive-ui';
</script>
<template>
    <n-grid x-gap="10" :y-gap="10" :cols="24" style="text-align: center">
        <n-gi :span="8" :offset="8">
            <n-grid
                x-gap="10"
                :y-gap="10"
                :cols="24"
                style="text-align: center"
            >
                <n-gi :span="24" class="canvas-box">
                    <canvas ref="canv" :width="0" :height="0"></canvas>
                </n-gi>
                <n-gi :span="18" :offset="3">
                    <n-upload
                        multiple
                        directory-dnd
                        accept="image/*"
                        :custom-request="customRequest"
                    >
                        <n-upload-dragger :multiple="false" style="width: 100%">
                            <div style="margin-bottom: 12px">
                                <n-icon size="48" :depth="3">
                                    <Archive></Archive>
                                </n-icon>
                            </div>
                            <n-text style="font-size: 16px">
                                点击或者拖动文件到该区域来生成像素图片
                            </n-text>
                            <n-p depth="3" style="margin: 8px 0 0 0">
                                请不要上传敏感数据，比如你的银行卡号和密码，信用卡号有效期和安全码
                            </n-p>
                        </n-upload-dragger>
                    </n-upload>
                </n-gi>
                <n-gi :span="18" :offset="3">
                    <n-slider
                        v-model:value="pixSize"
                        :step="1"
                        :min="1"
                        :max="max"
                        :on-update:value="sliderChange"
                    />
                    {{ pixSize }}
                </n-gi>
            </n-grid>
        </n-gi>
    </n-grid>
</template>

<script>
    export default {
        name: 'Converter',
        data() {
            return {
                pixSize: 5,
                min: 1,
                max: 1,
            };
        },
        methods: {
            async chooseImg(e) {
                const fileList = e.target.files;
                const file = fileList.length > 0 ? fileList[0] : null;
                if (file) {
                    const $image = await this.readImg(file);
                    if ($image) {
                        console.log('$image.width', $image.width);
                        if ($image.width || $image.height > 500) {
                            new useMessage().info('图片尺寸不得超过500', {
                                duration: 3000,
                            });
                            return false;
                        }
                        this.img = $image;
                        this.max =
                            $image.width < $image.height
                                ? $image.width
                                : $image.height;
                        this.drawToCanvas(this.img);
                    }
                }
            },
            drawToCanvas($image) {
                // 绘制图片
                const pixSize = this.pixSize;
                const width = $image.width;
                const height = $image.height;
                const canv = this.$refs.canv;
                canv.width = width;
                canv.height = height;
                const ctx = canv.getContext('2d');
                ctx.drawImage($image, 0, 0);
                const pixWidth = Math.ceil(width / pixSize);
                const pixHeight = Math.ceil(height / pixSize);
                console.log('pixHeight', pixHeight);
                let line = 0;
                const loop = (line) => {
                    for (let i = 0; i <= pixWidth; i++) {
                        const x = i * pixSize + Math.floor(pixSize / 2);
                        const y = line * pixSize + Math.floor(pixSize / 2);
                        const imageData = ctx.getImageData(
                            x < width ? x : width - pixSize,
                            y < height ? y : height - pixSize,
                            1,
                            1
                        );
                        const color = this.convertColor(imageData.data);
                        ctx.fillStyle = `rgba(${color.join(',')})`;
                        ctx.fillRect(
                            i * pixSize,
                            line * pixSize,
                            pixSize,
                            pixSize
                        );
                        if (i === pixWidth && line < pixHeight - 1) {
                            line++;
                            loop(line);
                        }
                    }
                };
                loop(line);
            },
            readImg(file) {
                return new Promise((resolve) => {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        const $image = new Image();
                        $image.onload = () => {
                            resolve($image);
                        };
                        $image.src = e.target.result;
                    };
                    reader.readAsDataURL(file);
                });
            },
            convertColor(imageData) {
                // const arr = [imageData[0], imageData[1], imageData[2]];
                // const max = Math.max(...arr);
                // const r = imageData[0] === max ? 255 : imageData[0];
                // const g = imageData[1] === max ? 255 : imageData[1];
                // const b = imageData[2] === max ? 255 : imageData[2];
                const r = imageData[0];
                const g = imageData[1];
                const b = imageData[2];
                const a = imageData[3];
                return [r, g, b, a];
            },
            async customRequest(e) {
                console.log(e);
                const $image = await this.readImg(e.file.file);
                if ($image) {
                    if ($image.width || $image.height > 500) {
                        const $message = useMessage();
                        $message.info('图片尺寸不得超过500', {
                            duration: 3000,
                        });
                        return false;
                    }
                    this.img = $image;
                    this.max =
                        $image.width < $image.height
                            ? $image.width
                            : $image.height;
                    this.drawToCanvas(this.img);
                }
            },
            sliderChange(val) {
                this.pixSize = val;
                if (this.img) {
                    this.drawToCanvas(this.img);
                }
            },
        },
    };
</script>

<style scoped>
    /deep/ .n-upload-trigger {
        width: 100%;
    }
    .canvas-box {
        display: flex;
        justify-content: center;
        align-content: center;
        height: 500px;
        width: 100%;
        border: 1px solid #dedede;
        overflow: hidden;
    }
</style>
