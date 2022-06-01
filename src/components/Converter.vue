<template>
    <n-grid :x-gap="50" :y-gap="10" :cols="24" style="text-align: center">
        <n-gi :span="14" class="canvas-box">
            <canvas ref="canv" :width="0" :height="0"></canvas>
        </n-gi>
        <n-gi
            :span="10"
            style="
                text-align: left;
                display: flex;
                justify-content: flex-start;
                align-items: center;
            "
        >
            <div>
                <n-form
                    label-placement="left"
                    label-width="auto"
                    require-mark-placement="right-hanging"
                >
                    <n-form-item label="上传图片：">
                        <n-upload
                            :show-file-list="false"
                            v-model:file-list="fileList"
                            @update:file-list="handleFileListChange"
                            directory-dnd
                            accept="image/*"
                            :custom-request="customRequest"
                        >
                            <n-button type="primary">
                                <template #icon>
                                    <n-icon>
                                        <Archive />
                                    </n-icon>
                                </template>
                                选择图片
                            </n-button>
                        </n-upload>
                        <span class="file-name">{{ img ? img.name : '' }}</span>
                    </n-form-item>
                    <n-form-item label="颜色：">
                        <n-radio-group
                            v-model:value="bit"
                            name="bit"
                            :on-update:value="
                                (val) => {
                                    bit = val;
                                    sliderChange();
                                }
                            "
                        >
                            <n-space>
                                <n-radio :key="1" :value="1">
                                    {{ 256 / 1 }}位
                                </n-radio>
                                <n-radio :key="4" :value="4">
                                    {{ 256 / 4 }}位
                                </n-radio>
                                <n-radio :key="8" :value="8">
                                    {{ 256 / 8 }}位
                                </n-radio>
                                <n-radio :key="16" :value="16">
                                    {{ 256 / 16 }}位
                                </n-radio>
                                <n-radio :key="32" :value="32">
                                    {{ 256 / 32 }}位
                                </n-radio>
                            </n-space>
                        </n-radio-group>
                    </n-form-item>
                    <n-form-item label="像素大小：">
                        <n-slider
                            :disabled="!img"
                            v-model:value="pixSize"
                            :step="1"
                            :min="min"
                            :max="max"
                            :on-update:value="
                                (val) => {
                                    pixSize = val;
                                    sliderChange();
                                }
                            "
                        />
                    </n-form-item>

                    <n-form-item label="R：">
                        <n-slider
                            :disabled="!img"
                            v-model:value="r"
                            :step="1"
                            :min="-100"
                            :max="100"
                            :on-update:value="
                                (val) => {
                                    r = val;
                                    sliderChange();
                                }
                            "
                        />
                    </n-form-item>
                    <n-form-item label="G：">
                        <n-slider
                            :disabled="!img"
                            v-model:value="g"
                            :step="1"
                            :min="-100"
                            :max="100"
                            :on-update:value="
                                (val) => {
                                    g = val;
                                    sliderChange();
                                }
                            "
                        />
                    </n-form-item>
                    <n-form-item label="B：">
                        <n-slider
                            :disabled="!img"
                            v-model:value="b"
                            :step="1"
                            :min="-100"
                            :max="100"
                            :on-update:value="
                                (val) => {
                                    b = val;
                                    sliderChange();
                                }
                            "
                        />
                    </n-form-item>
                    <n-form-item label=" ">
                        <n-button
                            secondary
                            type="tertiary"
                            size="small"
                            :disabled="!img"
                            @click="resetRgb"
                        >
                            复位
                        </n-button>
                    </n-form-item>
                </n-form>
            </div>
        </n-gi>
    </n-grid>
</template>

<script>
    import { Archive } from '@vicons/ionicons5';
    import { useMessage } from 'naive-ui';
    export default {
        name: 'Converter',
        setup() {
            let message = useMessage();
            return {
                message,
            };
        },
        components: {
            Archive,
        },
        data() {
            return {
                img: null,
                fileList: [],
                timmer: null,
                bit: 16,
                pixSize: 5,
                min: 1,
                max: 5,
                r: 0,
                g: 0,
                b: 0,
                marks: {
                    0: '0',
                },
            };
        },
        methods: {
            drawToCanvas($image) {
                // 绘制图片
                const pixSize = this.pixSize;
                let width = $image.width;
                let height = $image.height;
                const p = width / height;
                const canv = this.$refs.canv;
                const ctx = canv.getContext('2d');

                if (width > height) {
                    const scale = width / 500;
                    width = scale > 1 ? 500 : width;
                    height = scale > 1 ? height / scale : height;
                } else {
                    const scale = height / 500;
                    width = scale > 1 ? width / scale : width;
                    height = scale > 1 ? 500 : height;
                }
                canv.width = width;
                canv.height = height;
                ctx.drawImage($image, 0, 0, width, height);
                if (pixSize === 1) {
                    return false;
                }
                const pixWidth = Math.ceil(width / pixSize);
                const pixHeight = Math.ceil(height / pixSize);
                // console.log('pixHeight', pixHeight);
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

                        const myImageData = ctx.createImageData(
                            pixSize,
                            pixSize
                        );
                        myImageData.data.forEach((item, index) => {
                            if ((index + 1) % 4 === 0) {
                                myImageData.data[index] = color[3];
                            } else {
                                myImageData.data[index] =
                                    color[((index + 1) % 4) - 1];
                            }
                        });
                        ctx.putImageData(
                            myImageData,
                            i * pixSize,
                            line * pixSize
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
                        $image.onload = (res) => {
                            setTimeout(() => {
                                resolve($image);
                            }, 50);
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
                const limitColor = (num) => {
                    if (num < 0) {
                        return 0;
                    } else if (num > 255) {
                        return 255;
                    } else {
                        return num;
                    }
                };
                const bit = this.bit;
                const r = limitColor(
                    Math.ceil(imageData[0] / bit) * bit + (this.r / 100) * 255
                );
                const g = limitColor(
                    Math.ceil(imageData[1] / bit) * bit + (this.g / 100) * 255
                );
                const b = limitColor(
                    Math.ceil(imageData[2] / bit) * bit + (this.b / 100) * 255
                );
                const a = imageData[3];
                return [r, g, b, a];
            },
            handleFileListChange(fileList) {
                console.log('handleFileListChange', fileList);
                this.fileList =
                    fileList.length > 0 ? [fileList[fileList.length - 1]] : [];
            },
            async customRequest(e) {
                console.log('customRequest', e);
                const $image = await this.readImg(e.file.file);
                $image.name = e.file.name;
                console.log('$image', $image);
                if ($image) {
                    if ($image.width > 5000 || $image.height > 5000) {
                        message.info('图片尺寸不得超过5000', {
                            duration: 3000,
                        });
                        return false;
                    }
                    this.img = $image;
                    this.max =
                        $image.width > $image.height
                            ? $image.width
                            : $image.height;
                    this.drawToCanvas(this.img);
                }
            },
            sliderChange(val) {
                if (this.timmer) {
                    clearTimeout(this.timmer);
                }
                this.timmer = setTimeout(() => {
                    if (this.img) {
                        this.drawToCanvas(this.img);
                    }
                }, 200);
            },
            resetRgb() {
                this.r = 0;
                this.g = 0;
                this.b = 0;
                this.drawToCanvas(this.img);
            },
        },
    };
</script>

<style scoped>
    :deep() .n-upload-trigger {
        width: 100%;
    }
    .canvas-box {
        display: flex;
        justify-content: center;
        align-content: center;
        height: 500px;
        width: 100%;
        border: 1px solid #63e2b7;
        overflow: hidden;
    }
    .file-name {
        display: inline-block;
        text-overflow: hidden;
        overflow: hidden;
        word-break: break-all;
        white-space: nowrap;
    }
</style>
