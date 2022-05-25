<template>
    <n-grid x-gap="10" :y-gap="10" :cols="24" style="text-align: center">
        <n-gi :span="24">
            <n-grid
                x-gap="10"
                :y-gap="10"
                :cols="24"
                style="text-align: center"
            >
                <n-gi :span="6" :offset="3" class="canvas-box">
                    <canvas ref="canv1" :width="0" :height="0"></canvas>
                </n-gi>
                <n-gi :span="6" :offset="3" class="canvas-box">
                    <canvas ref="canv2" :width="0" :height="0"></canvas>
                </n-gi>
                <n-gi :span="18" :offset="3">
                    <n-upload
                        :show-file-list="false"
                        v-model:file-list="fileList"
                        directory-dnd
                        accept="image/*"
                        :custom-request="customRequest"
                    >
                        <n-upload-dragger style="width: 100%">
                            <div style="margin-bottom: 12px">
                                <n-icon size="32" :depth="3">
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
                <n-gi :span="9" :offset="3">
                    <n-form-item label-placement="left" label="颜色">
                        <n-radio-group v-model:value="bit" name="bit" :on-update:value="(val) => { bit = val; sliderChange() }">
                            <n-space>
                                <n-radio :key="8" :value="8">
                                    {{ 8 }}位
                                </n-radio>
                                <n-radio :key="16" :value="16">
                                    {{ 16 }}位
                                </n-radio>
                                <n-radio :key="32" :value="32">
                                    {{ 32 }}位
                                </n-radio>
                                <n-radio :key="64" :value="64">
                                    {{ 64 }}位
                                </n-radio>
                            </n-space>
                        </n-radio-group>
                    </n-form-item>
                </n-gi>
                <n-gi :span="9">
                    <n-form-item label-placement="left" label="像素大小">
                        <n-slider
                            v-model:value="pixSize"
                            :step="1"
                            :min="min"
                            :max="max"
                            :on-update:value="(val) => { pixSize = val; sliderChange() }"
                        />
                    </n-form-item>
                </n-gi>
            </n-grid>
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
                message
            }
            
        },
        components: {
            Archive,
        },
        data() {
            return {
                fileList: [],
                timmer: null,
                bit: 16,
                pixSize: 10,
                min: 5,
                max: 5,
                r: 0,
                g: 0,
                b: 0
            };
        },
        methods: {
            drawToOrignal($image) {
                const pixSize = this.pixSize;
                const width = $image.width;
                const height = $image.height;
                const canv = this.$refs.canv1;
                canv.width = width;
                canv.height = height;
                const ctx = canv.getContext('2d');
                ctx.drawImage($image, 0, 0);
            },
            drawToCanvas($image) {
                // 绘制图片
                const pixSize = this.pixSize;
                const width = $image.width;
                const height = $image.height;
                const canv = this.$refs.canv2;
                canv.width = width;
                canv.height = height;
                const ctx = canv.getContext('2d');
                ctx.drawImage($image, 0, 0);
                const pixWidth = Math.ceil(width / pixSize);
                const pixHeight = Math.ceil(height / pixSize);
                // console.log('pixHeight', pixHeight);
                let line = 0;

                const imageDataArr = [];
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
                        imageDataArr.push(color);

                        // TODO 重新写入像素
                        const myImageData = ctx.createImageData(pixSize, pixSize);
                        myImageData.data.forEach((item, index) => {
                            if((index + 1)%4 === 0) {
                                myImageData.data[index] = color[3];
                            } else {
                                myImageData.data[index] = color[(index + 1)%4 - 1];
                            }
                        });
                        ctx.putImageData(myImageData, i * pixSize, line * pixSize);
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

                const bit = this.bit;
                const r = Math.ceil(imageData[0]/bit) * bit + this.r;
                const g = Math.ceil(imageData[1]/bit) * bit + this.g;
                const b = Math.ceil(imageData[2]/bit) * bit + this.b;
                const a = imageData[3];
                return [r, g, b, a];
            },
            async customRequest(e) {
                console.log(this.fileList);
                const $image = await this.readImg(e.file.file);
                if ($image) {
                    if ($image.width > 5000 || $image.height > 5000) {
                        message.info('图片尺寸不得超过500', {
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
                    this.drawToOrignal(this.img);
                }
            },
            sliderChange(val) {
                if(this.timmer) {
                    clearTimeout(this.timmer);
                }
                this.timmer = setTimeout(() => {
                    if (this.img) {
                        this.drawToCanvas(this.img);
                    }
                }, 200)
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
        border: 1px solid #dedede;
        overflow: hidden;
    }
</style>
