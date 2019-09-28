<template>
  <v-layout>
    <v-flex
      xs12
      sm8
      md6
    >
      <v-row align="center">
        <v-col class="text-center" cols="12" sm="4">
          <video autoplay playsinline muted id="wc" width="224" height="224"></video>
        </v-col>
      </v-row>

      <v-row align="center" v-if="isTrainigShown">
        <v-col class="text-center" cols="12" sm="4">
          <v-btn large color="primary" v-on:click="handleButton('0')" :disabled="notReady">Healthy</v-btn>
          <v-btn large color="secondary" v-on:click="handleButton('1')" :disabled="notReady">Sick</v-btn>
          <v-btn large color="error" v-on:click="handleButton('2')" :disabled="notReady">Dying</v-btn>
        </v-col>
      </v-row>

      <v-row align="center" v-if="isTrainigShown">
        <v-col class="text-center" cols="12" sm="4">
          <div id="healthysamples">Healthy Samples: {{ healthysamples }}</div>
          <div id="sicksamples">Sick Samples: {{ sicksamples }}</div>
          <div id="dyingamples">Dying Samples: {{ dyingamples }}</div>
        </v-col>
      </v-row>

      <v-row align="center" v-if="isTrainigShown">
        <v-col class="text-center" cols="12" sm="4">
          <v-btn large color="primary" v-on:click="train()" :disabled="notReady">Train Network</v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col align="center" cols="12" sm="4">
          <v-btn large color="primary" v-on:click="startPredicting()" :disabled="cantPredict">
            Start Predicting
          </v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="text-center" cols="12" sm="4">
          <v-btn large color="primary" v-on:click="save()" :disabled="cantPredict" v-if="isTrainigShown">
            Save
          </v-btn>
          <v-btn large color="primary" v-on:click="showTraining()" v-if="!isTrainigShown">
            Show Training
          </v-btn>
          <v-btn large color="secondary" v-on:click="load()" :disabled="!cantPredict">
            Load
          </v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="text-center" cols="12" sm="4">
          <h2>{{predictionText}}</h2>
        </v-col>
      </v-row>
      <!--TODO <button type="button" id="stopPredicting" onclick="stopPredicting()">Stop Predicting</button>-->
    </v-flex>
  </v-layout>
</template>
<script>
  const samples = 5
  const modelUrl = 'indexeddb://lukas-health-model'
  import Webcam from '~/components/webcam.js'
  import RPSDataset from '~/components/rpsDataset.js'

  export default {
    components: {
      Webcam,
      RPSDataset
    },
    head() {
      return {
        title: this.title,
        meta: [
          { hid: 'description', name: 'description', content: 'My custom description' }
        ],
        script: [
          { src: 'https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest' }
        ]
      }
    },
    data() {
      return {
        webcam: {},
        dataset: {},
        mobilenet: {},
        healthysamples: 0,
        sicksamples: 0,
        dyingamples: 0,
        notReady: true,
        isPredicting: false,
        predictionText: '',
        cantPredict: true,
        isTrainigShown: false
      }
    },

    async mounted() {
      let model
      let mobilenet
      const webcam = new Webcam(document.getElementById('wc'))
      const dataset = new RPSDataset()
      this.webcam = webcam
      this.dataset = dataset

      // reshape mobi net
      async function loadMobilenet() {
        const mobilenet = await tf.loadLayersModel('https://storage.googleapis.com/tfjs-models/tfjs/mobilenet_v1_0.25_224/model.json')
        const layer = mobilenet.getLayer('conv_pw_13_relu')
        return tf.model({ inputs: mobilenet.inputs, outputs: layer.output })
      }

      async function init() {
        await webcam.setup()
        mobilenet = await loadMobilenet()
        tf.tidy(() => mobilenet.predict(webcam.capture()))
        return {
          mobilenet
        }
      }

      const afterInit = await init()
      this.mobilenet = afterInit.mobilenet
      this.notReady = false
      console.warn('mounted done')
    },
    methods: {
      train: async function () {
        this.notReady = true
        this.dataset.ys = null
        this.dataset.encodeLabels(3)
        this.model = tf.sequential({
          layers: [
            tf.layers.flatten({ inputShape: this.mobilenet.outputs[0].shape.slice(1) }),
            tf.layers.dense({ units: 100, activation: 'relu' }),
            tf.layers.dense({ units: 3, activation: 'softmax' })
          ]
        })
        const optimizer = tf.train.adam(0.0001)
        this.model.compile({ optimizer, loss: 'categoricalCrossentropy' })
        let loss = 0

        await this.model.fit(this.dataset.xs, this.dataset.ys, {
          epochs: 10,
          callbacks: {
            onBatchEnd: async (batch, logs) => {
              loss = logs.loss.toFixed(5)
              console.log('LOSS: ' + loss)
            }
          }
        })
        this.notReady = false
        this.cantPredict = false
      },
      handleButton: function (id) {
        switch (id) {
          case '0':
            this.healthysamples += samples
            break
          case '1':
            this.sicksamples += samples
            break
          case '2':
            this.dyingamples += samples
            break
        }

        let label = parseInt(id)
        for (let i = 0; i < samples; i++) {
          const img = this.webcam.capture()
          this.dataset.addExample(this.mobilenet.predict(img), label)
        }
      },
      startPredicting: function () {
        this.isPredicting = true
        this.predict()
      },
      stopPredicting: function () {
        this.isPredicting = false // TODO check on navigation out
        this.predict()
      },
      save: async function () {
        await this.model.save(modelUrl)
        this.stopPredicting()
      },
      showTraining: function() {
        this.isTrainigShown = true
      },
      load: async function () {
        this.model = await tf.loadLayersModel(modelUrl)
        this.cantPredict = false
        this.notReady = false
      },
      predict: async function () {
        while (this.isPredicting) {
          const predictedClass = tf.tidy(() => {
            const img = this.webcam.capture()
            const activation = this.mobilenet.predict(img)
            const predictions = this.model.predict(activation)
            console.log(predictions.as1D().argMax())
            console.log(predictions.as1D().max())

            return predictions.as1D().argMax()
          })

          const classId = (await predictedClass.data())[0]
          let predictionText = '' // TODO here we can set statefull
          switch (classId) {
            case 0:
              predictionText = 'I see you are healthy'
              break
            case 1:
              predictionText = 'I see are getting sick'
              break
            case 2:
              predictionText = 'I see you need to get medical attention ASAP'
              break
            case -1:
              predictionText = 'How do you feel today?'
              break
          }
          this.predictionText = predictionText

          predictedClass.dispose()
          await tf.nextFrame()
        }
      }
    }

  }
</script>
