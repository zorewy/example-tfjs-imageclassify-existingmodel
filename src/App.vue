<template>
  <div id="app">
    <div>
      <img id="img" src="/coffee.jpg" :width="imgSize" :height="imgSize" />
    </div>
    <Button @click="onPredict" :disabled="isRunning || !isLoadModel">预测</Button>
    <div v-html="text"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import * as tf from '@tensorflow/tfjs'
import * as MobileNet from '@tensorflow-models/mobilenet'

@Component
export default class App extends Vue {
  // 图片尺寸
  imgSize = 224
  // 模型是否已加载
  isLoadModel = false
  // 正在预测
  isRunning = false
  // 结果
  text = ''

  // 图像分类模型
  model: tf.LayersModel | any

  mounted () {
    this.text = '正在加载模型...'
    // 使用cpu
    tf.setBackend('cpu')
    // 加载模型
    MobileNet.load()
        .then(mobileNet => {
          this.model = mobileNet
          this.isLoadModel = true
          this.text = '模型加载成功'
        })
        .catch(e => {
          console.error(e)
          this.text = e.message
        })
  }

  onPredict () {
    if (!this.isLoadModel) {
      alert('模型加载失败，无法进行预测')
      return
    } else if (this.isRunning) {
      alert('当前正在预测中')
      return
    }
    this.predict()
  }

  /**
   * 预测
   */
  predict () {
    this.isRunning = true
    this.text = '正在预测...'
    const img: any = document.getElementById('img')
    // 调用模型进行预测,取出前5个
    this.model.classify(img, 5)
        .then((result: Array<any>) => {
          // 输出
          const items: Array<string> = []
          result.forEach((item: any) => items.push(`${item.className}：${Math.round(100 * item.probability)}%`))
          this.text = items.join('<br>')
        })
        .catch((e: any) => this.text = e)
        .finally(() => this.isRunning = false)
  }
}
</script>
