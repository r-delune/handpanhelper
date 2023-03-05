<template>
  <div>
    <v-container>
      <v-row no-gutters>
        <h1 style="font-size: 32px" class="pr-4">Handpan Helper</h1>
        <v-select disabled label="Root note" v-model="rootNote"></v-select>
        <!-- enter handpan note -->
        <v-combobox
          label="Available notes on your handpan"
          :items="allNotes"
          multiple
          v-model="myHandpanNotes"
          v-on:change="reformulateHandpanNotesAsNotes()"
        ></v-combobox>
        <!-- {{ handPanNotesAsNumbers }} -->
      </v-row>
      <v-row v-if="myHandpanNotes.length > 0">
        <!-- show chord types -->
        <!-- <v-container> -->
        <v-row>
          <v-col class="pa-5">
            <div class="pa-2" v-if="hasMinor !== false">
              Minor: {{ hasMinor.status }}
              <div :key="i" v-for="i in hasMinor.notes">
                <div :class="i.color">{{ i.note }}</div>
              </div>
            </div>

            <div class="pa-2" v-if="hasMajor !== false">
              Major: {{ hasMajor.status }}
              <div :key="i" v-for="i in hasMajor.notes">
                <div :class="i.color">{{ i.note }}</div>
              </div>
            </div>
            <!-- <v-spacer></v-spacer> -->
            <v-btn class="" @click="record()">Record a note</v-btn>
          </v-col>
          <v-col v-if="myHandpanNotes.length > 0">
            <div id="app" class="pa-1">
              <svg :width="paths.svgSize" :height="paths.svgSize">
                <path
                  :key="index"
                  :id="`path${index}`"
                  v-on:click="pathClick(index)"
                  :d="paths.d"
                  :transform="transform"
                  :fill="paths.colors[index]"
                  stroke-width="4"
                  stroke="white"
                  v-for="(transform, index) in paths.transforms"
                ></path>
                <text
                  :key="index"
                  v-on:click="pathClick(index)"
                  :transform="transform"
                  fill="white"
                  stroke="white"
                  :xlink:href="`#path${index}`"
                  v-for="(transform, index) in paths.textTransforms"
                >
                  <tspan dx="59">{{ paths.sections[index] }}</tspan>
                </text>
              </svg>
            </div>
          </v-col>
        </v-row>
        <!-- </v-container> -->
      </v-row>
      <!-- <v-row v-else>
        <h1 style="font-size: 132px" class="pa-4">Handpan Helper</h1>
      </v-row> -->

      <!-- <v-btn @click="startPitchDetecter()">startPitchDetect</v-btn> -->
    </v-container>
  </div>
</template>
<script>
import { ref, computed } from "vue"
export default {
  name: "IndexPage",
  setup() {
    return {
      currentNotes: "None",
      majorNotes: ref(true),
      minorClass: ref("bg-red-lighten-3 px-2"),
      majorClass: ref("bg-red-lighten-3 px-2"),
      minorNotes: ref(true),
      myHandpanNotes: ref([]),
      majorIntervals: [1, 5, 8],
      minorIntervals: [1, 2, 8],
      hasMinor: ref(false),
      hasMajor: ref(false),
      majorSeventhIntervals: [1, 2, 8],
      domininantIntervals: [1, 5, 8],
      subDomninant: [1, 5, 8],
      first: ref(1),
      rootNote: ref("C"),
      context: null,
      allNotesAsNumbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
      allNotes: ["C", "C♯", "D", "D♯", "E", "F", "F♯", "G", "G♯", "A", "A♯", "B"],
      handPanNotesAsNumbers: ref([]),
      myHandPan: [""],
      svgSize: 420,
      colors: [
        "#EF476F",
        "#FFD166",
        "#456990",
        "#06D6A0",
        "#118AB2",
        "#073B4C",
        "#028090",
        "#F45B69",
      ],
    }
  },

  mounted() {
    this.context = new AudioContext()
    // this.setupMic();
  },
  watch: {
    myHandpanNotes() {
      // reconfigure all notes once they have been chosen
      this.checkAvailableChords()
    },
  },

  methods: {
    checkAvailableChords() {
      this.handPanNotesAsNumbers = this.convertHandpanNotesToValues()
      this.hasMinor = this.checkIfTheseChordsArePossible(this.minorIntervals)
      this.hasMajor = this.checkIfTheseChordsArePossible(this.majorIntervals)
    },
    checkIfTheseChordsArePossible(scale) {
      let json = {
        color: "bg-light-blue-lighten-3 px-2",
        notes: [],
      }

      if (scale.every((elem) => this.handPanNotesAsNumbers.includes(elem))) {
        json.status = "Has all elements of this chord"
        for (var i = 0; i < scale.length; i++) {
          json.notes.push({ note: this.convertSingleValueToNotes(scale[i]), color: "green" })
        }
        return json
        // check if some elements exist
      } else if (scale.some((elem) => this.handPanNotesAsNumbers.includes(elem))) {
        json.status = "Has some elements of this chord"
        for (var i = 0; i < scale.length; i++) {
          let convertedScaleNumberToNote = this.convertSingleValueToNotes(scale[i])
          if (this.myHandpanNotes.includes(convertedScaleNumberToNote)) {
            json.notes.push({
              note: this.convertSingleValueToNotes(scale[i]) + " (interval " + scale[i] + ")",
              color: "bg-light-blue-lighten-3 px-2",
            })
          } else {
            json.notes.push({
              note: this.convertSingleValueToNotes(scale[i]) + " (interval " + scale[i] + ")",
              color: "bg-red-lighten-3 px-2",
            })
          }
        }
        return json
      } else {
        json.status = "You do not have any notes of this chord"
        for (var i = 0; i < scale.length - 1; i++) {
          json.notes.push({
            note: this.convertSingleValueToNotes(scale[i]) + " (interval " + scale[i] + ")",
            color: "bg-red-lighten-3 px-2",
          })
        }
        return json
      }
    },
    convertHandpanNotesToValues() {
      let arr = []
      for (var i in this.myHandpanNotes) {
        arr.push(this.allNotes.indexOf(this.myHandpanNotes[i]) + 1)
      }
      return JSON.parse(JSON.stringify(arr))
    },
    convertSingleValueToNotes(note) {
      let thisnote = note - 1
      return this.allNotes[thisnote]
    },
    convertValuesToNotes(notes) {
      let arr = []
      let tempArr = this.allNotes
      for (let i = 0; i < notes.length; i++) {
        let val = tempArr[notes[i] - 1]
        arr.push(val)
      }
      return arr
    },
    coordinatesFromDegree(degree, radius) {
      const angleinRadians = degree * (Math.PI / 180)
      const y = Math.round(Math.sin(angleinRadians) * radius)
      const x = Math.round(Math.cos(angleinRadians) * radius)
      return { x, y }
    },
    // record from mic - turn into note values
    async record() {
      const { PitchDetector } = await import("pitchy")
      navigator.mediaDevices
        .getUserMedia({ audio: true })
        .then(function (localStream) {
          var audioContext = new (window.AudioContext || window.webkitAudioContext)()
          var input = audioContext.createMediaStreamSource(localStream)
          var analyser = audioContext.createAnalyser()
          var scriptProcessor = audioContext.createScriptProcessor()
          // Some analyser setup
          analyser.smoothingTimeConstant = 0
          analyser.fftSize = 64

          input.connect(analyser)
          analyser.connect(scriptProcessor)
          scriptProcessor.connect(audioContext.destination)
          var getAverageVolume = function (array) {
            var length = array.length
            var values = 0
            var i = 0
            for (; i < length; i++) {
              values += array[i]
            }
            return values / length
          }
          var onAudio = function () {
            var tempArray = new window.Uint8Array(analyser.frequencyBinCount)
            analyser.getByteFrequencyData(tempArray)
            var latestFrequency = getAverageVolume(tempArray)
            //use latestFrequency
            // console.log(latestFrequency);
            // console.log(PitchDetector);
            // PitchDetector.findPitch(latestFrequency, 2048);
          }
          scriptProcessor.onaudioprocess = onAudio
        })
        .catch(function () {
          //Handle error
        })
    },
    // setup the microphone
    setupMic() {
      return
      var source
      var analyser = this.context.createAnalyser()
      analyser.minDecibels = -100
      analyser.maxDecibels = -10
      analyser.smoothingTimeConstant = 0.85
      navigator.getUserMedia(
        { audio: true },
        (stream) => {
          this.context.createMediaStreamSource(stream).connect(analyser)

          const dataArray = new Uint8Array(analyser.frequencyBinCount)

          analyser.getByteTimeDomainData(dataArray)

          // Log the contents of the analyzer ever 500ms.
          setInterval(() => {
            console.log(dataArray.length)
          }, 500)
        },
        (err) => console.log(err)
      )
    },
    // pitch detect - turn into note values
    startPitchDetecter() {
      const voice = new Wad({ source: "mic" }) // At this point, your browser will ask for permission to access your microphone.
      const tuner = new Wad.Poly()
      tuner.setVolume(0) // If you're not using headphones, you can eliminate microphone feedback by muting the output from the tuner.
      tuner.add(voice)
      voice.play() // You must give your browser permission to access your microphone before calling play().
      tuner.updatePitch() // The tuner is now calculating the pitch and note name of its input 60 times per second. These values are stored in <code>tuner.pitch</code> and <code>tuner.noteName</code>.
      const logPitch = function () {
        console.log(tuner, tuner.noteName)
        requestAnimationFrame(logPitch)
      }
      logPitch()
      tuner.stopUpdatingPitch() // Stop calculating the pitch if you don't need to know it anymore.
    },
    pathClick(arg) {
      console.log("apth click", arg, this.myHandpanNotes[arg])

      let freq = parseFloat(this.getFrequency(this.myHandpanNotes[arg] + "4")) // make it the third octave
      // console.log("converted freq", freq, this.myHandpanNotes[arg])
      this.playNote(freq)
    },
    playNote(freq) {
      freq = freq
      var vco = this.context.createOscillator()
      vco.type = vco.SINE
      vco.frequency.value = freq
      vco.start(0)
      var vca = this.context.createGain()
      vca.gain.value = 0
      vco.connect(vca)
      vca.connect(this.context.destination)
      vco.frequency.value = freq
      vca.gain.value = 1

      setTimeout(function () {
        vco.stop(0)
        console.log("stopping")
      }, 1000)
    },
    getFrequency(note) {
      var notes = ["A", "A#", "B", "C", "C#", "D", "D#", "E", "F", "F#", "G", "G#"],
        octave,
        keyNumber

      if (note.length === 3) {
        octave = note.charAt(2)
      } else {
        octave = note.charAt(1)
      }

      keyNumber = notes.indexOf(note.slice(0, -1))

      if (keyNumber < 3) {
        keyNumber = keyNumber + 12 + (octave - 1) * 12 + 1
      } else {
        keyNumber = keyNumber + (octave - 1) * 12 + 1
      }

      // Return frequency of note
      return 440 * Math.pow(2, (keyNumber - 49) / 12)
    },
  },
  computed: {
    paths() {
      const svgSize = this.svgSize
      const midSvgSize = svgSize / 2
      const diameter = 400 // Should be even no
      const r = 200 //  diameter / 2
      const divisions = this.handPanNotesAsNumbers.length // Should be a factor of 360
      const degree = 360 / divisions
      const coords = this.coordinatesFromDegree(degree, r)
      const d = `M 0,0 l ${r},0 A ${r},${r},0,0,0,${coords.x},${-coords.y} L 0,0`
      const transforms = []
      const textTransforms = []
      const colors = this.colors
      const sections = this.myHandpanNotes

      for (let i = 0; i < divisions; i += 1) {
        transforms.push(`rotate (${i * degree + degree / parseFloat(2)}
          ${midSvgSize} ${midSvgSize}) translate (${midSvgSize}, ${midSvgSize})`)
        textTransforms.push(` rotate(${i * degree} ${midSvgSize} ${midSvgSize})
          translate(${midSvgSize}, ${midSvgSize})`)
      }
      return {
        svgSize,
        d,
        transforms,
        textTransforms,
        colors,
        sections,
      }
    },
  },
}
</script>
<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 420px;
  margin: 0 auto;
}
</style>
